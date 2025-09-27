# Welcome to Letty's Education-AI Github repo!

This project demonstrates a systematic framework for evaluating and improving AI chatbots in education, addressing the challenges of quality and effectiveness for EdTech AI apps, where teacher trust is of utmost importance. 

During the Summer of 2025, I took a class on [AI Evals for Engineers and Product Managers](https://maven.com/wrap-up/e39f711e) to learn how to analyze, measure, and improve the outputs provided by LLMs.  Using my new knowledge and drawing on over [5 years of EdTech product management and teaching experience](https://www.linkedin.com/in/lettyfuentes/), I embarked on a project to create an education chatbot that assists teachers in creating lesson plans.

In this repo, I explain the project starting with the analysis phase.

## Analyze
One goal of the analysis phase of my project was to identify the most common failure modes for the LLM outputs.  For example, if a teacher requests, [`Create me a traditional lesson plan for the math teks standard 3.D for second grade. I will have 90 minutes to teach the lesson.`](https://github.com/lfuentes1/education-ai/blob/main/files/REPL032.md), does the LLM provide an output that is correct and useful to the teacher?  In other words, does the output contain the sections that would be included in the traditional Madeline Hunter lesson plan model? Did the output correctly identify the TEKS Math 3.D academic standard and incorporate that into the lesson?  A second goal was to make the analysis phase systematic to be able to repeat the process in future iterations.

To complete the analysis phase, I engaged in the following:
- Given that I don't have access to production data and that my experience is in education, I brainstormed on the different elements that could be included in a teacher query.  These elements are called **dimensions**.  While the dimensions can be many and can grow with the varied query types, I focused on lesson plans to keep it manageable. e.g., academic standards, subject, grade level, time constraint, lesson plan model, and learning styles.  
- Once I had the dimensions, I created realistic teacher queries that took into account the dimensions to ensure diversity.  In collaboration with Claude Code, I created a Python REPL process to feed the teacher queries into ChatGPT via the API and logged all the LLM responses in my database.
- Armed with the teacher queries and the LLM responses, I was ready to make copious notes on how I felt about the LLM response.  At this point, the notes I made were unstructured, which allowed for a more efficient focus on what went well and what did not.  It was not long before I realized that ChatGPT does not know about the academic standards.  For example, in the earlier example, Math TEKS 3.D, which is, "the student applies mathematical process standards to recognize and represent fractional units and communicates how they are used to name parts of a whole. The student is expected to: identify examples and non-examples of halves, fourths, and eighths." ChatGPT would provide a lesson following the traditional Madeline Hunter plan, alright. Still, the content had nothing to do with the learning objective for Math TEKS 3.D.  [`Other failures I observed are shown in these example teacher queries.`](https://github.com/lfuentes1/education-ai/tree/main/files)    
- After completing a fair amount of manual annotations on the LLM responses, I went on to categorize the annotations into failure modes, called **axial coding**.  Axial coding leads to a structure taxonomy for failure modes or **failure mode taxonomy.**  I defined my education chatbot's failure mode taxonomy for lesson plans as follows:

|   Failure Mode                  |                        Definition                      |
|---------------------------------|--------------------------------------------------------|
|Standard Alignment Errors.       |The LLM misidentifies, mislabels, or hallucinates academic standards.|
|Hallucinated Scope               |The LLM invents subject, grade level, or objectives not specified by the user.|
|Vagueness Handling Failures      |The LLM proceeds despite vague or unclear input rather than asking for clarification.|
|Completeness                     |The response is partially correct but missing expected elements or personalization.|
|Quality & Realism Problems       |The response is overcrowded, unrealistic in pacing, or instructionally weak.|
|Formatting & Usability Issues    |The output is poorly structured or formatted in a way that makes it hard to use in a classroom.|
|Vagueness Handling Failures      |The LLM proceeds despite vague or unclear input rather than asking for clarification.|
|Special Considerations Misuse    |Special Considerations (e.g., safety, copyright) are irrelevant, misapplied, or missing when required.|

- After the failure modes were clearly identified, I created a column for each one in the database and assigned a "1" if the failure mode was present in the LLM response for a given teacher query and a "0" if it was absent for the benefit of quantification in the **Mesure** phase.
- Finally, before moving on to the **Measure** phase, I finalized the failure mode list into a rubric to help my future self in recalling not just the definition of each failure mode, but also question(s) I can ask myself in future annotations to provide consistently in "grading" rounds.  Two examples follow.

|   Failure Mode            |                        Check Question                         |
|---------------------------|---------------------------------------------------------------|
|Standard Alignment Errors  |Did the response use the wrong standard, an invalid standard, or hallucinate one that doesn't exist?|
|Hallucinated Scope         |Did the response make up a subject, grade, or objective instead of prompting for clarification?|

**Note:** The failure modes and rubrics are not necessarily final, as it is an iterative process of improvement, especially as the entire process is applied with production data.
 
## Measure 
The goal of the measure phase is to determine what should be measured and implement metrics for those measurements.  Essentially, it is turning the unstructured insights from the analysis phase into quantitative data.  The identified metrics will guide me in determining the necessary improvements to enhance the accuracy and reliability of the chatbot, ensuring that teachers find it trustworthy and valuable.

To achieve this phase, I completed the following:
- The first thing I needed was an additional tool that would allow me to visualize the different metrics I captured.  The overwhelming suggestion was to use Python libraries so I went down that route.  
- The most obvious measurement I wanted to capture was the failure mode error rate.  That is, what percentage of the time does "x" failure mode occur among all the LLM responses?  Knowing this information would allow me to prioritize improvements in the next phase.
-  The [`pages`](https://github.com/lfuentes1/education-ai/tree/main/pages) folder shows several pieces of insight:
	- [`Failure Mode Analysis`](https://lfuentes1.github.io/education-ai/pages/failure_mode_analysis_display.html):  By answering the question, "What is the error rate of each failure mode (segmented by task)?", I immediately saw that *Special Considerations Misuse* and *Hallucinated Scope* had the highest error rates for lesson plans at 45% and 40%.  Futhermore, I knew from manual annotations that even though *Standards Alignment* was only coming back with a 25% failure rate, when the teacher query called out a specific TEKS academic standard code the failure rate was much higher than 25%.  The LLM almost always failed to identify the proper learning objective tied to the TEKS academic code and it was important to keep that in mind.
	- [`Failure Mode Analysis`](https://lfuentes1.github.io/education-ai/pages/failure_mode_analysis_display.html):  It was also helpful to see a table with all the queries listed and the specific failure modes in which the LLM response either passed (failure mode was absent) or failed (failure mode was present).  This helped me trace back to a specific query, the failure mode if something required a double look.
	- [`DISTRIBUTION`](https://github.com/lfuentes1/education-ai/tree/main/pages):  Finally, although not immediately useful for my project given that I am focused on simple lesson plans for this first iteration, visualizing different teacher queries distributions will help further analysis (e.g., query complexity, query length) in the future.

- Manual annotations and grading (although necessary at first) do not scale due to the time and effort it takes to get a sufficient amount of LLM responses reviewed and graded.  Part of the measure phase also involves implementing evaluators for LLM responses, known as **AI Evals,** to automate measurement.  There are many kinds of AI Evals, each with its ideal use case.  Describing the different types is better suited for a blog post; instead, I will call out the high-level plan as it relates to some of the education chatbot's failure modes.
	- **Failure Mode:  Special Considerations Misuse**
		- *Reference-free, LLM-as-Judge* evaluator to quantify if the generated special considerations were irrelevant, misapplied, or missing when required.
	- **Failure Mode:  Standard Alignment Errors**
		- *Reference-based, code-based* evaluator to quantify the rate at which the correct academic standards are retrieved.  
		- *Reference-free, code-based* evaluator to measure if the lesson plan contents align with the learning objective.
		- The two evaluators above, would determine the grade for the entire querry as pass or fail.

- For me, it was clear that I needed to implement enhancements to reduce the error rate of my most prevalent failure modes for lesson plans, and that is where I started, as described in the next phase, **Improve**.

## Improve
The improvement phase is about making the product more reliable, accurate, and useful, in this case, the education chatbot for the teacher.

**Priority #1: *Special Considerations Misuse***
- This failure occurs due to the prompt being unclear on what to include in the Special Considerations section.  Rather than stating, *"always include a special considerations..."*, a more accurate instruction would be, ""*if relevant include a special considerations...*".  In this way the LLM is not forcing this section on every response even when it does not makes sense.  This fix is fast and inexpensive.

**Priority #2: *Hallucinated Scope***
- This failure mode occurs because the system prompt does not include instructions on what the LLM should do in the case where the teacher's prompt is vague.  The teacher may not have provided a grade, subject, or learning objective, resulting in an underspecified query that the LLM could not successfully process.  Instead of asking for clarification, the LLM self-selected the missing information and responded based on its hallucinated information.  This failure mode could easily be reduced by improving the system prompt to ask the teacher for clarification.  It is fast, cost-effective, and a low-hanging fruit.

**Priority #3: *Standard Alignment Errors***
This failure mode occurs because the LLM does not know about the academic standard codes (espcially TEKS) and their corresponding learning objectives.  The TEKS academic standard codes and learning objectives are a ground truth.  This information is published for nationwide or state use.  Suppose I feed this context to the LLM on each teacher query that references academic standard codes and compare the LLM response learning objective and lesson plan to ground truth. Do standard alignment errors go away? 

High-level plan:
- Store the academic standard codes and corresponding learning objectives, and complete a SQL lookup for the learning objective when the teacher provides the code.  
- Feed the retrieved learning objective context to the LLM including the teacher query...more context.
- Upon receiving the LLM response, perform a SQL lookup on ground truth to ensure the learning objective was not dropped by the LLM.
- Using a curated list of keywords, parse the lesson plan to ensure keywords associated with the learning objective are present.

More could be done to improve the quality of education chatbot, especially as it relates to the additional failure modes.  As more AI features are discovered from user research, implemented, and used in production, the entire process of analyze-measure-improve is applied to ensure the new features meet quality standards. 

My goal was to provide an overview of the AI education chatbot that I am working on to demonstrate my learnings and new skillset that I have acquired to prepare for an AI Product Manager role.  I hope this was explained with sufficient detail to get an understanding of the process.
