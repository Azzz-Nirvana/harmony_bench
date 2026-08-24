# harmony_bench

Tool package repository.

## Structure

- `dist/bin/` - Executable files
- `dist/configs/` - Configuration files
- `dist/README.md` - Original README

## Large Files

Files over 100MB are split into parts:
- `run_eval.exe` (246 MB) -> `run_eval.exe.part1`, `run_eval.exe.part2`, `run_eval.exe.part3`
- `arkts_ast_bridge.exe` (102 MB) -> `arkts_ast_bridge.exe.part1`, `arkts_ast_bridge.exe.part2`

To restore: `copy /b run_eval.exe.part1+run_eval.exe.part2+run_eval.exe.part3 run_eval.exe`