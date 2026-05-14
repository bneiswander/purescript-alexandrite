## 1. Fix instance-chain blocking in matching

- [x] 1.1 Update `non_determined_unification_ids` in `compiler-core/checking/src/core/constraint/matching.rs` to skip arguments whose match result is not unknown (i.e., skip `Match` and `Apart` results, only collect from `Stuck`/`Skolem`)
- [x] 1.2 Add regression fixture `1778720160_example_function_fundep` with a minimal `Example` class and instance chain that reproduces the false-positive pattern
- [x] 1.3 Run `cargo check -p checking --tests` to verify the crate compiles
- [x] 1.4 Run related instance-chain/fundep fixtures to guard against regressions: `1772301720_instance_constraint_solving`, `1772301780_instance_functional_dependency`, `1778051640_open_row_instance_chain_apart`, `1778704620_else_instance_open_row_match`
- [x] 1.5 Accept the new fixture snapshot with `just t checking 1778720160_example_function_fundep --accept`
- [x] 1.6 Build and install the compiler with `cargo build -p purescript-analyzer && cargo install --path compiler-bin --force`
