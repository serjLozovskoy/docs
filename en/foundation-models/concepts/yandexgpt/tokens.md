# Tokens

Neural networks work with texts by representing words and sentences as _tokens_. Tokens are small pieces of information or frequent character sequences that are common for a natural language. Tokens help neural networks detect patterns and process natural language.

{{ yagpt-full-name }} uses its own tokenizer for text processing. You can estimate the text field size in tokens using the [`Tokenizer`](../..//text-generation/api-ref/Tokenizer/index.md) REST API methods. The token count of the same text may vary from one [model](models) to another.