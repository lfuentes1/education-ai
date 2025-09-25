# Welcome to Letty's Education-AI Github repo!

This project demonstrates a systematic framework for evaluating and improving AI chatbots in education, addressing the challenges of quality and effectiveness for EdTech AI apps, where teacher trust is of utmost importance. 

During the Summer of 2025, I took a class on [AI Evals for Engineers and Product Managers](https://maven.com/wrap-up/e39f711e) to learn how to analyze, measure, and improve the outputs provided by LLMs.  Using my new knowledge and drawing on over 5 years of EdTech product management experience, I embarked on a project to create an education chatbot that assists teachers in creating lesson plans.

In this repo, I explain the project starting with the analysis phase.

## Analyze
One goal of the analysis phase of my project was to identify the most common failure modes for the LLM outputs.  For example, if a teacher requests, "Create me a lesson plan for TEKS Math K.2A that follows the traditional Madeline Hunter model," does the LLM provide an output that is correct and useful to the teacher?  In other words, does the output contain the sections that would be included in the traditional Madeline Hunter lesson plan model?  Did the output correctly identify the TEKS Math K.2A academic standard and incorporate that into the lesson?  A second goal was to make the analysis phase systematic to be able to repeat the process in future iterations.

To complete the analysis phase, I engaged in the following:
- Given that I don't have access to production data and that my experience is in education, I brainstormed on the different elements that could be included in a teacher query.  The class calls these elements **dimensions**.  While the dimensions can be many and can grow with the varied query types, I focused on lesson plans to keep it manageable. e.g., academic standards, subject, grade level, time constraint, lesson plan model, and learning styles.  
- Once I had the dimensions, I created realistic teacher queries that took into account the dimensions to ensure diversity.  In collaboration with Claude Code, I created a batch process to feed the teacher queries into ChatGPT via the API and logged all the LLM responses in my database.
- Armed with the teacher queries and the LLM responses, I was ready to make copious notes on how I felt about the LLM response.  At this point, the notes I made were unstructured, which allowed for a more efficient focus on what went well and what did not.  It was not long before I realized that ChatGPT does not know about the academic standards.  For example, in the earlier example, TEKS K.2A, which is about teaching to "count forward and backward to at least 20 with and without objects," ChatGPT would provide a lesson following the traditional Madeline Hunter plan, alright. Still, the content had nothing to do with the learning objective of "counting forward and backward...."  Other failures I observed included those related to ChatGPT hallucinating on the scope of the query (asked to give a Math lesson without specifics, it proceeded with providing a Math lesson of its own choice without asking for clarification on the scope).    
- After completing a fair amount of annotations on the LLM responses, I went on to categorize the annotations into failure modes, called **axial coding**.  Axial coding leads to a structure taxonomy for failure modes or **failure mode taxonomy.**  I defined my education chatbot's failure mode taxonomy for lesson plans as follows:

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

- After the failure modes were clearly identified, I created a column for each one in the database and assigned a "1" if the failure mode was present in the LLM response for a given teacher query and a "0" if it was absent.
- Finally, before moving on to the **Measure** phase, I finalized the failure mode list into a rubric to help my future self in recalling not just the definition of each failure mode, but also question(s) I can ask myself in future annotations to provide consistently in "grading" rounds.  Two examples follow.

|   Failure Mode            |                        Check Question                         |
|---------------------------|---------------------------------------------------------------|
|Standard Alignment Errors  |Did the response use the wrong standard, an invalid standard, or hallucinate one that doesn't exist?|
|Hallucinated Scope         |Did the response make up a subject, grade, or objective instead of prompting for clarification?|

**Note:** The failure modes and rubrics are not necessarily final, as it is an iterative process of improvement, especially as the entire process is applied with production data.
 
## Measure 
The goal of the measure phase is to determine what should be measured and implement metrics for those measurements.  Essentially, it is turning the unstructured insights from the analysis phase into qualitative data.  The identified metrics will guide me in determining the necessary improvements to enhance the accuracy and reliability of the chatbot, ensuring that teachers find it trustworthy and valuable.

To achieve this phase, I completed the following:
- The first thing I needed was an additional tool that would allow me to visualize the different metrics I wanted to capture.  This would allow me to visualize my data in a more efficient and fun way!  The overwhelming suggestion was to use Python libraries so I went down that route.  
- The most obvious measurement I wanted to capture was the failure mode error rate.  That is, what percentage of the time does "x" failure mode occur among all the LLM responses?  Knowing this information would allow me to prioritize improvements in the next phase.
-  The [`pages/failure_mode_analysis_display.html`](https://lfuentes1.github.io/education-ai/pages/failure_mode_analysis_display.html) file shows several pieces of insight:
	- By answering the question, "What is the error rate of each failure mode?", I immediately saw that *Standards Alignment* and *Hallucinated Scope* had the highest error rates.  This measurement aligned with my observations as I manually annotated the LLM responses during the analysis phase.  
	- It was also helpful to see a table with all the queries listed and the specific failure modes in which the LLM response either passed (failure mode was absent) or failed (failure mode was present).  This helped me trace back to a specific query, the failure mode if something required a double look.

- Manual annotations and grading (although necessary at first) do not scale due to the time and effort it takes to get a sufficient amount of LLM responses reviewed.  Part of the measure phase also involves implementing automated evaluators for LLM responses, known as **AI Evals.**  There are many kinds of AI Evals, each with its ideal use case.  Describing the different types is better suited for a blog post; instead, I will call out the high-level plan as it relates to some of the education chatbot's failure modes.
	- **Failure Mode:  Standard Alignment Errors**
		- *Reference-based, code-based* evaluator to look up the learning objective and provide the context to the LLM.  Upon the LLM response, compare the objective returned to the ground truth.  Finally, use a *reference-free, code-based* evaluator to verify that the lesson plan contents align with the learning objective.  Grade the entire trace as pass or fail.
	- **Failure Mode:  Completeness**
		- *Reference-free, code-based* evaluator to check that the LLM output has all the required elements based on the teacher query.  Grade the entire trace as pass or fail.
 	- **Failure Mode:  Special Considerations Misuse**
	 	- *Reference-free, LLM-as-Judge* evaluator to verify if the generated special considerations were irrelevant, misapplied, or missing when required.
	
- For me, it was clear that I needed to implement enhancements to reduce the error rate of my most prevalent failure modes, and that is where I started, as described in the next phase, **Improve**.

## Improve
The improvement phase is about making the product more reliable, accurate, and useful, in this case, the education chatbot for the teacher.

**Priority #1: *Standard Alignment Errors***
This failure mode occurs because the LLM does not know about the academic not know about the academic standard codes and their corresponding learning objectives.  The academic standard codes and learning objectives are a ground truth.  This information is published for nationwide or state use.  Suppose I feed this context to the LLM on each teacher query that references academic standard codes and compare the LLM response learning objective and lesson plan to ground truth. Do standard alignment errors go away? 

High-level plan (in progress):
- Store the academic standard codes and corresponding learning objectives, and complete a SQL lookup for the learning objective when the teacher provides the code.  
- Feed the retrieved learning objective context to the LLM including the teacher query.
- Upon receiving the LLM response, perform a SQL lookup to ensure the learning objective was not dropped by the LLM.
- Using a curated list of keywords, parse the lesson plan to ensure keywords associated with the learning objective are present.

**Priority #2: *Hallucinated Scope***
- This failure mode occurs because the system prompt does not include instructions on what the LLM should do in the case where the teacher's prompt is vague.  The teacher may not have provided a grade, subject, or learning objective, resulting in an underspecified query that the LLM could not successfully process.  Instead of asking for clarification, the LLM self-selected the missing information and responded based on its hallucinated information.  This failure mode could easily be reduced by improving the system prompt.  It is fast, cost-effective, and a low-hanging fruit.

**Priority #3: *Suboptimal Quality***
- This failure mode occurs when the response provided by the LLM is overcrowded, unrealistic in pacing, or instructionally weak.  Note:  I need an improvement plan.

**Priority #4: *Vagueness Handling***
- This failure mode occurs because the LLM proceeds with providing a response despite the teacher query being vague or unclear rather than asking for clarfiication.  Similar to the hallucinated scope failure mode, this failure mode could be reduced by improving the system prompt, which is fast and inexpensive.

More could be done to improve the quality of education chatbot, especially as it relates to the additional failure modes.  As more AI features are discovered from user research, implemented, and used in production, the entire process of analyze-measure-improve is applied to ensure the new features meet quality standards. 

My goal was to provide an overview of the AI education chatbot that I am working on to demonstrate my learnings and new skillset that I have acquired to prepare for an AI Product Manager role.  I hope this was explained with sufficient detail to get an understanding of the process.
