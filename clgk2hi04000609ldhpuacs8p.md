---
title: "Task Management system for langchain"
seoTitle: "Task Management system for langchain - python demo and project code"
seoDescription: "How to use/build a task management system system in python for use with the langchain library. Designed for assisting LLMs (tested with GPT-3.5-turbo)."
datePublished: Sun Apr 16 2023 23:57:42 GMT+0000 (Coordinated Universal Time)
cuid: clgk2hi04000609ldhpuacs8p
slug: task-management-system-for-langchain
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1681689375512/b7c967e1-aecf-46f2-8428-5acd77ee9d1b.png
tags: python, automation, langchain

---

[View project on Replit](https://replit.com/@MarcusWeinberger/ai-task-manager)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681689146990/2d670ae8-6ebe-446c-9baa-e24097c290d4.png align="center")

# How does it work?

The `TaskManager` gets created, with a **goal**, a list of **tools**, and its own **LLM**. It includes a bunch of prompts which is used internally. When initialized, it creates an initial list of tasks to get started (this behaviour is different when initializing with certain arguments, but I'll get into that later).

As a developer, it's up to you how you process those tasks, but you can see what I would do at the end of [`main.py`](http://main.py). But, as a task is completed, you give the TaskManager the name of the task and the result. The `TaskManager` will then use another prompt to reflect on the results of this task. This is called the **refinement** process. If additional tasks are needed, they will be added here. Info that may be needed for future tasks is saved to `stored_info`, and stuff for the end result *should* go to `final_result`.

When the final goal has been met, the `TaskManager` will run the `complete_func` (which you can define in the initialization) which will - by default - save all variables to a file.

## Cool features

* Internal LLM to fix JSON output
    
* Can persist state
    
* Uses `text-davinci-003` or something by default (cheap)