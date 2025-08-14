### Greenfield agentic flow

1. (Optional) Start with the [product exploration](product-exploration.md) prompt to explore your idea. Discuss the behavior and features on this level. Don't focus on technology yet.

2. Discuss your project with a large thinking model using [project document compilation](project-doc-compilation.md) prompt. Apply corrections until you're happy with the outcome. This is the step where tech stack decisions should be taken.

3. Feed the compiled project document from step 2 into the [component analysis](component-analysis.md) prompt again using a large thinking model. AI will generate a document containing self-contained modules that can be fed to an agent like Cursor, Cline, Claude Code, Codex or Gemini CLI. Iterate with the AI until you are happy with the shape of these document.

4. Put the product description and project document into the agent's context. These documents describe what the product does and how it should be built. Add a style guide and rule sets of the dependencies that you use.

5. Start feeding modules from the component analysis document as tasks to the agent. Include relevant parts of the product description, project document and the style guide into the context. It's good to help AI shape the APIs as you see how the code looks like in practice at this level. You can also leverage the ask mode to analyze some unclear details before triggering agentic implementation.

6. Iterate until built! 