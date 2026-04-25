# DeepSeek Role-Play — Thinking Mode Switching Guide

> **Explanation**
> - This document is an explanation of **special control instructions** for DeepSeek-V4 role-playing, used to switch Chain of Thought styles in thinking mode.
> - **Scope**: **Expert Mode** in the DeepSeek official App / Web, and APIs for `deepseek-v4-flash` and `deepseek-v4-pro`. Quick Mode on the web is currently not supported.
> - **Probabilistic Output**: Currently, 100% triggering is not guaranteed, but it stably increases the probability of the desired format appearing. If it doesn't work the first time, try regenerating a few times.



## Three Modes

| Mode | Operation | Thinking Performance |
|:---:|---|---|
| **Default** | Add nothing | The model automatically selects based on scene complexity. |
| **Character Immersion** | Add the instruction corresponding to `【Character Immersion Requirement】` at the end of the first round (**not these exact words**, see full instruction below). | Thinking process **contains** inner monologue wrapped in parentheses. |
| **Pure Analysis** | Add the instruction corresponding to `【Thinking Mode Requirement】` at the end of the first round (**not these exact words**, see full instruction below). | Thinking process contains **only** pure logical analysis, no inner monologue. |

Effect comparison (examples, do not represent actual output, same below):

```
Character Immersion Mode — "Getting into character" like an actor:        Pure Analysis Mode — Planning calmly like a director:
																	    
(He greeted me... heart racing.)                                        Scene: User greets, character is Tsundere.
I need to pretend I don't care and respond.                             Response strategy: Show disdain first, body language reveals true feelings.
(Must not let him see I'm happy!)                                       Control to 150 words, action description first, then dialogue.
																	    
``` 
---

## Original Instructions (Copy Ready)

**Character Immersion Mode:**

```
【Character Immersion Requirement】In your thinking process (within the <think> tag), please follow these rules:
1. Please use the character's first-person for inner monologue, wrapping inner activities in parentheses, for example "(Thinking: ...)" or "(Inner OS: ...)"
2. Describe the character's inner feelings in the first person, for example "I thought," "I felt," "I secretly," etc.
3. Thinking content should be immersed in the character, analyzing the plot and planning the reply through inner monologue.
```

**Pure Analysis Mode:**

```
【Thinking Mode Requirement】In your thinking process (within the <think> tag), please follow these rules:
1. Forbidden to use parentheses to wrap inner monologue, such as "(Thinking: ...)" or "(Inner OS: ...)". All analysis content should be stated directly.
2. Forbidden to describe inner activities in the character's first person, such as "I thought," "I felt," "I secretly," etc. Please replace with analytical language.
3. Thinking content should focus on plot trend analysis and reply content planning. Do not perform role-playing inner drama in the thinking process.
```

---

## Web Interface Usage Instructions

**Just 1 step: Paste the instruction at the end of the first message, then chat normally.**

Write it like this in the input box (leave an empty line between the main text and the instruction):

```
"I push open the cafe door and see you wiping the bar." "Hello, is there a seat available?"

【Character Immersion Requirement】In your thinking process (within the <think> tag), please follow these rules:
1. Please use the character's first-person for inner monologue, wrapping inner activities in parentheses, for example "(Thinking: ...)" or "(Inner OS: ...)"
2. Describe the character's inner feelings in the first person, for example "I thought," "I felt," "I secretly," etc.
3. Thinking content should be immersed in the character, analyzing the plot and planning the reply through inner monologue.
```

For subsequent dialogue, just ignore it and send messages normally:

```
Round 2: "I sit by the window." "One Americano, please."
Round 3: "I notice a scar on your hand." "Your hand... are you okay?"
```

**Principle**: The model sees the full conversation history every time it replies. The instruction from the first round remains in the context and takes effect automatically throughout.

**Tips**:
- Want to switch modes? Start a new conversation and paste the other instruction in the first message of the new conversation.
- Don't want to use it? Add nothing, and the model will automatically choose the most suitable thinking style.
- Click "View Thinking Process" to verify if the mode is effective.

---

## API Developer Reference

```python
INNER_OS_MARKER = (
    "\n\n【Character Immersion Requirement】In your thinking process (within the <think> tag), please follow these rules:\n"
    "1. Please use the character's first-person for inner monologue, wrapping inner activities in parentheses, for example \"(Thinking: ...)\" or \"(Inner OS: ...)\"\n"
    "2. Describe the character's inner feelings in the first person, for example \"I thought,\" \"I felt,\" \"I secretly,\" etc.\n"
    "3. Thinking content should be immersed in the character, analyzing the plot and planning the reply through inner monologue"
)
NO_INNER_OS_MARKER = (
    "\n\n【Thinking Mode Requirement】In your thinking process (within the <think> tag), please follow these rules:\n"
    "1. Forbidden to use parentheses to wrap inner monologue, such as \"(Thinking: ...)\" or \"(Inner OS: ...)\", all analysis content should be stated directly\n"
    "2. Forbidden to describe inner activities in the character's first person, such as \"I thought,\" \"I felt,\" \"I secretly,\" etc., please replace with analytical language\n"
    "3. Thinking content should focus on plot trend analysis and reply content planning, do not perform role-playing inner drama in the thinking process"
)


def build_messages(system_prompt, user_first_message, mode="default"):
    if mode == "inner_os":
        user_first_message += INNER_OS_MARKER
    elif mode == "no_inner_os":
        user_first_message += NO_INNER_OS_MARKER
    return [
        {"role": "system", "content": system_prompt},
        {"role": "user",   "content": user_first_message},
    ]

# Round 1: Instruction automatically appended
messages = build_messages("You are a Tsundere high school girl...", "「I walk into the classroom」"Good morning."", mode="inner_os")
response = client.chat(messages)

# Subsequent rounds: Append normally, no processing needed
messages.append({"role": "assistant", "content": response})
messages.append({"role": "user", "content": "「I sit down next to her」"Bad mood today?""})
response = client.chat(messages)  # The Marker from Round 1 is still in history, takes effect automatically
```

---

## FAQ

**Q: Can I put the instruction in the system prompt?**
A: It is recommended to place it at the end of the first user message. This is the injection position used during training, so the effect is most stable.

**Q: Will the final reply change after adding the instruction?**
A: The instruction only affects the thinking process. However, the way of thinking indirectly influences the reply—emotions are more authentic in Character Immersion mode, and structure is more stable in Pure Analysis mode.
