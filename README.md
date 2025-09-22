# Welcome to Letty's Github!

I am a Product Manager transitioning into AI Product Management.  During the Summer of 2025, I took a class on [AI Evals for Engineers and Product Managers](https://maven.com/wrap-up/e39f711e) to learn how to analyze, measure, and improve the outputs provided by LLMs.  I had heard great things about the class and was not disappointed.  

In this repo, I highlight my learnings and the project that I worked on to practice what I learned in class, as well as independent learning that I have engaged in to enhance the material learned in class.  Given that I have over 5 years of experience in education technology (EdTech), this project focuses on education, specifically an education chatbot.

Project Description:  COMPLETE

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
