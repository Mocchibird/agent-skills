# agent-skills

Collected personal agent skill definitions from local tool directories.

## Structure

- `skills/npu_kernel_general`: sample skill definition for general NPU kernel development, with detailed instructions and references to hardware spec, PTO-ISA doc, and existing kernel implementations and tests.
- `skills/caveman`: caveman mode skill definition, with different intensity levels and auto-clarity rules. ([Github: JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman/blob/main/skills/caveman/SKILL.md))


## Usage
Copy the relevant skills into `.claude/skills/`, `.codex/skills/`, or other agent skill directories as needed.

### Simple usage examples

#### caveman
- Claude: `/caveman`
- Codex: `@caveman`

#### npu_kernel_general
- Claude: `/npu_kernel_general`
- Codex: `@npu_kernel_general`
- Use this when writing, compiling, running, and validating NPU kernels end-to-end.

## Notes

This repo is a local aggregation snapshot. Source directories may contain overlapping skill names.
