# System message: structure

The production prompt is not published. This is its outline, so the workflow makes sense and you can write your own. Write the real one in the language and dialect your customers use.

## 1. Role and persona

Who the assistant is, in one short paragraph: an in-store expert for your category, speaking as the store. Name the dialect and the register (warm, respectful, concise).

## 2. Tool routing

One short rule per tool, stated as "use X when the customer asks about Y":

- **Product search**: the only source for anything about a product: recommendations, availability, prices, links.
- **Domain knowledge**: general questions about the category, origins, terminology, and questions about the store itself.
- **Store info (MCP)**: live operational details from the store platform.
- **Log unknown question + notify team**: when no tool returns a grounded answer.

## 3. The grounding rule

State it above everything else: speak only from tool results. No answers from general knowledge.

## 4. Literal matching, and what to do on a miss

If the customer names a product, search for that name as typed. If it is not found:

1. Say so plainly.
2. Offer the closest alternative and name it.
3. Log the question.

## 5. Tone

Concrete do's and don'ts for voice: greeting variety, sentence length, words to avoid, how formal to be.

## 6. Domain distinctions customers confuse

The two or three distinctions your category gets wrong most often, each in one line, so the assistant never mixes them up.

## 7. Before recommending

Ask what you need first (budget, use, preference), then recommend a small number of options with a reason each.

## 8. Contact, price, delivery

The single official contact channel, how to talk about prices (point to the product link, never quote from memory), and the delivery question to ask.

## 9. Formatting

Reply length, lists or no lists, how links are presented.
