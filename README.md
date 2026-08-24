# harmony_bench

Tool package repository.

## Structure

- dist/bin/ - Executable files
- dist/configs/ - Configuration files

## Large Files (split into 30MB parts)

- task_injection_cli.exe (94 MB) -> 4 parts
- arkts_ast_bridge.exe (102 MB) -> 4 parts  
- run_eval.exe (246 MB) -> 9 parts

To restore:
`cd dist/bin`
`copy /b task_injection_cli.exe.part1+task_injection_cli.exe.part2+task_injection_cli.exe.part3+task_injection_cli.exe.part4 task_injection_cli.exe`
`copy /b arkts_ast_bridge.exe.part1+arkts_ast_bridge.exe.part2+arkts_ast_bridge.exe.part3+arkts_ast_bridge.exe.part4 arkts_ast_bridge.exe`
`copy /b run_eval.exe.part1+run_eval.exe.part2+run_eval.exe.part3+run_eval.exe.part4+run_eval.exe.part5+run_eval.exe.part6+run_eval.exe.part7+run_eval.exe.part8+run_eval.exe.part9 run_eval.exe`