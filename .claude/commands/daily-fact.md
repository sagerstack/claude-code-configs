---
name: daily-fact
description: Get current time, generate a random number, and share an interesting fact about that number
---

## Instructions
Execute the following subagents in sequence:

1. First, invoke the timekeeper subagent to get the current date and time
2. Next, invoke the random-number-generator subagent to generate a random number between 1-1000
3. Finally, take the number from step 2 and invoke the number-fact-generator subagent with that number to generate an interesting fact

Present all three results in a clear, organized format showing the current time, the random number generated, and the interesting fact about that number.