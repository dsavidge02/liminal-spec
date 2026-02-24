## Model Selection

Different models have different strengths. Choosing the right model for a task improves both quality and efficiency.

**General principles:**

- **Capable models** work well for implementation, orchestration, and writing. They handle context, synthesize across documents, and make pragmatic decisions.
- **Detail-oriented models** work well for verification, code review, and spec auditing. They catch what builders miss -- signature mismatches, spec drift, edge cases.
- **Using multiple models** for validation provides complementary coverage. Author + different model catches more issues than author alone.

Leave specific model choice to the practitioner. The principle matters more than the specific tool: match model strengths to task requirements.
