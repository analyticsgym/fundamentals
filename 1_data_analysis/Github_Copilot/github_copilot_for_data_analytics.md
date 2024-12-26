# Github Copilot for Data Analytics Coding

### Objective
- Notes on high-level best practices for using Github Copilot to write SQL, R, or Python code for data analytics.
- Github Copilot is a pair programming AI tool that can help you write code faster and learn new angles/methods for data analysis.

### Tips
#### 1. Context building
- Context inputs set the stage for how the AI can help and learn about your problem space/objective.
- Ways the AI can gain context: objective statements at the top of code, code comments, sample data, target output, data details, other open files (if enabled), variable naming, function names, etc.

#### 2. Break down complex tasks into small tasks
- Review and iterate on suggested code to guide the AI towards the goal destination.
- Suggestions will get truncated if the AI is given too much to do at once (due to compute resource constraints). 

#### 3. Different ways to prompt
- Use code comments that describe a goal for what a code chunk should do.
- Descriptive naming of functions, variables, etc which the AI can use for inference and auto-completion suggestion.
- `# q`: used to prompt the AI with a question inline (note: complex questions better suited for AI chat interface at this time).
- `# a`: response to a question prompt.

#### 4. Stay intellectually rigorous
- Not all AI suggestions are correct or best practice.
- Said differently, don't blindly accept the AI suggestions and hope for the best (similar to using Stack Overflow answers).

#### 5. Use the AI as a learning tool
- Prompt the AI to write code in a different way, leverage different packages, suggest different angles to solve a problem, write X code in a more efficient way, etc.

### Github Copilot + GenAI Chat Tools
- GenAI chat tools like ChatGPT, Gemni, etc can be used alongside Github Copilot to help with more complex questions, problem solving, collaboration, etc.
- Currently, the chat tools offer more robust discussion and code suggestions.
- Several tools exist to bring the chat interface to your coding environment (e.g. VSCode, JupyterLab, Rstudio, etc). 
- Tends to require API access to the GenAI chat tool.
- Testing TBD (Rstudio + chattr, JupyterLab + Jupyter AI, etc).
- GenAI chat interface can be used outside of the coding environment if API access not available.












