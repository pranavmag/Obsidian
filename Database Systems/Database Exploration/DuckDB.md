2026-09-26 01:41

Tags: 

Execution is split across two folders. Operators and physical plan generation live in `src/execution/`, but the `Executor` that schedules and runs pipelines lives in `src/parallel/`.

### Query | Logical and Physical Plan

```
CREATE TABLE t AS SELECT range AS a FROM range(10);
SET explain_output = 'all';
EXPLAIN SELECT a FROM t WHERE a > 5;
```

So here I'm creating a table named t that takes a range of values 0 - 9 as a. I set it to 'all' for explain_output which means that both the optimized logical plan and physical plan are going to be showed in the output for a query. So I want it DuckDB to explain that for selecting the values of a from t where a is greater than 5.

```
bash-5.3$ ./build/debug/duckdb
DuckDB v2.0.0-dev85809 (Development Version, d8a1bd4f4f)
Enter ".help" for usage hints.
memory D CREATE TABLE t AS SELECT range AS a FROM range(10);
memory D SET explain_output = 'all';
memory D EXPLAIN SELECT a FROM t WHERE a > 5;
╭─ Projection ───────────────────────╮
│ Expressions: a                     │
╰──────────────────┬─────────────────╯
╭─ Filter ─────────┴─────────────────╮
│ Expressions: a > CAST(5 AS BIGINT) │
╰──────────────────┬─────────────────╯
╭─ Seq Scan ───────┴─────────────────╮
│ Table: memory.main.t               │
│ Type: Sequential Scan              │
╰────────────────────────────────────╯

╭─ Projection ──────────╮
│ Expressions: a        │
│ ~2 rows               │
╰───────────┬───────────╯
╭─ Seq Scan ┴───────────╮
│ Filters: a > 5        │
│ Table: memory.main.t  │
│ Type: Sequential Scan │
│ ~2 rows               │
╰───────────────────────╯

╭─ Seq Scan ────────────╮
│ Table: memory.main.t  │
│ Type: Sequential Scan │
│ Projections: a        │
│ Filters: a > 5        │
│ ~2 rows               │
╰───────────────────────╯

(END)
```

```
1. Unoptimized logical      2. Optimized logical       3. Physical

Projection (a)              Projection (a)  ~2 rows    Seq Scan
    │                           │                        Projections: a
Filter (a > CAST(5 AS BIGINT))     Seq Scan                     Filters: a > 5
    │                         Filters: a > 5              ~2 rows
Seq Scan (t)                  ~2 rows
```

So this is what the EXPLAIN output shows us. Plan 1 is exactly like how our SQL is starting with the sequential scan (FROM t), then going into the filter (WHERE a > 5), and the projection (SELECT a). The planner builds this naive tree. I see that we do a CAST here. I searched it up and found that range() returns BIGINT values which is a 64-bit signed integer so it sees that '5' is mismatched since it is an integer literal. The binder inserts the cast, because the binder resolves names against the catalog and resolves types.

For the optimized logical plan I see that there was a filter push down which now sits inside the sequential scan. This makes sense intuitively because we go from reading all 10 rows to discarding 6 of them before the sequential scan happens. Also constant folding happened here, the cast now just became 5, the otpimizer computed it once and just replaced the expression with its result. I also see a cardinality estimate, here is is ~2 rows which is a bit off but that's likely because of a default selectivity like 20% of 10 rows is 2. My question here to look into more deeply later will be **"Why did the optimizer estimate 2 rows?"**

The physical plan also moved the projection down as well. Based off my textbook reading, I know that in most cases it is optimal to push selections and projections down so this makes sense to me. So everything essentially collapsed into a single operator by the end.

### Debugger | Logical and Physical Plan

In the debugger, moving through the execution I reached `logical_planner.CreatePlan(std::move(statement));` and this shows the binding + plan and there is also `logical_plan = optimizer.Optimize(std::move(logical_plan));` which shows that optimize is being called to create the optimized logical plan. We later also see `result->physical_plan = physical_planner.Plan(std::move(logical_plan));` which turns the logical plan into a physical plan. This matches what I saw earlier when I used that query.

One thing I want to note down here is that the code looks like each phase consumes the plan and returns a new one. The code uses std::move(logical_plan) which gives ownership of the whole tree to the next phase, which then returns its own version. `plan = Phase(std::move(plan))` is probably a good way to put it and I think that it's a good and easy to read pipeline.

Also another thing I noticed is that we can turn off optimization if needed.

```
bool optimize = Settings::Get<EnableOptimizerSetting>(*this);

if (Settings::Get<DebugDisableOptimizerSetting>(*this)) {

// verify disable optimizer - disable EXCEPT for explain, otherwise every single EXPLAIN query breaks

if (logical_plan->type != LogicalOperatorType::LOGICAL_EXPLAIN) {

optimize = false;

}
```

I think the next thing I might try now is to see how the pipeline is like if I disable optimizations when I write a query.

```
bash-5.3$ ./build/debug/duckdb
DuckDB v2.0.0-dev85809 (Development Version, d8a1bd4f4f)
Enter ".help" for usage hints.
memory D PRAGMA disable_optimizer;
┌─────────┐
│ Success │
│ boolean │
└─────────┘
  0 rows 
memory D CREATE TABLE t AS SELECT range AS a FROM range(10);
memory D SET explain_output = 'all';
memory D EXPLAIN SELECT a FROM t WHERE a > 5;
╭─ Projection ───────────────────────╮
│ Expressions: a                     │
╰──────────────────┬─────────────────╯
╭─ Filter ─────────┴─────────────────╮
│ Expressions: a > CAST(5 AS BIGINT) │
╰──────────────────┬─────────────────╯
╭─ Seq Scan ───────┴─────────────────╮
│ Table: memory.main.t               │
│ Type: Sequential Scan              │
╰────────────────────────────────────╯

╭─ Projection ───────────────────────╮
│ Expressions: a                     │
╰──────────────────┬─────────────────╯
╭─ Filter ─────────┴─────────────────╮
│ Expressions: a > CAST(5 AS BIGINT) │
╰──────────────────┬─────────────────╯
╭─ Seq Scan ───────┴─────────────────╮
│ Table: memory.main.t               │
│ Type: Sequential Scan              │
╰────────────────────────────────────╯

╭─ Filter ──────────────────────────╮
│ Expression: a > CAST(5 AS BIGINT) │
│ ~0 rows                           │
╰─────────────────┬─────────────────╯
╭─ Seq Scan ──────┴─────────────────╮
│ Table: memory.main.t              │
│ Type: Sequential Scan             │
│ ~0 rows                           │
╰───────────────────────────────────╯

(END)
```

Plans 1 and 2 are now the exact same since I turned off optimizations. Earlier the 2nd plan moved the filter down into the sequential scan and also applied constant folding on the cast expression, but now it's passed on to the next phase unoptimized. So this affects the physical plan in the end as well because now the filter happens after the sequential scan. 

I think this also tells us a couple other interesting things as well. The projection is still put into the sequential scan in the physical plan so that means that part was a physical plan generator thing and not an optimizer thing. Also the cardinality estimations changed from ~2 to ~0 so that definitely tells us that cardinality estimation is the job of the optimizer.

### Parser

```
ShellState::ExecuteSQL
 → ClientContext::IterateStatements
   → StatementIterator(ParseIterator)
     → Peek(): tokenize all once (EnsureTokenized), then parse one statement per call
```

Full Complete Path based off what I've looked into 

```
PARSE      ClientContext::IterateStatements → ParseIterator::Peek
           (tokenize once, parse one statement per call)
BIND+PLAN  Planner::CreatePlan → binder->Bind          planner.cpp
OPTIMIZE   Optimizer::Optimize                          optimizer.cpp
PHYSICAL   PhysicalPlanGenerator::Plan                  physical_plan_generator.cpp
           (all three called from ClientContext::CreatePreparedStatementInternal)
EXECUTE    ClientContext::SubmitPreparedStatementInternal
           → Executor::Initialize(result collector)     executor.cpp
           → results pulled via ExecuteTaskInternal / WaitForTask
```

### Optimizer


