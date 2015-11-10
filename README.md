# liblex
C library for Lexical Analysis

[![Join the chat at https://gitter.im/lwci/liblex](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/lwci/liblex?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


#### Usage:

```c
#include <stdio.h>
#include <liblex.h>

enum { /* Tokens */
	MYLEXER_PLUS = 1,
	MYLEXER_MINUS,
	MYLEXER_DIV,
	MYLEXER_MULT,
	MYLEXER_MOD,
	MYLEXER_START_COMMENT,
	MYLEXER_END_COMMENT,
	MYLEXER_WHITESPACE,
	MYLEXER_NUMBER,
	MYLEXER_IGNORE
};

enum { /* States */
	INITIAL = 0,
	COMMENT
};

int start_comment_callback(llex *lex, unsigned char *str, size_t len) 
{
	llex_set_state(lex, COMMENT);
	return MYLEXER_START_COMMENT;
}

int end_comment_callback(llex *lex, unsigned char *str, size_t len) 
{
	llex_set_state(lex, INITIAL);
	return MYLEXER_END_COMMENT;
}

int number_callback(llex *lex, unsigned char *str, size_t len) 
{
	return MYLEXER_NUMBER;
}

int main(int argc, char **argv)
{
	llex lex;
	llex_token_id token_id;
		
	llex_init(&lex);
	llex_set_buffer(&lex, "1 - 2 + 3 / 4 /* ignored str */");
	
	llex_set_state(&lex, INITIAL);
	llex_add_token_callback(&lex, "/*", start_comment_callback);
	
	llex_set_state(&lex, COMMENT);
	llex_add_token_callback(&lex, "*/", end_comment_callback);
	llex_add_token_regex(&lex, "(?:(?!\\*/).)+", MYLEXER_IGNORE);
	
	llex_set_state(&lex, INITIAL);
	llex_add_token(&lex, "+", MYLEXER_PLUS);
	llex_add_token(&lex, "-", MYLEXER_MINUS);
	llex_add_token(&lex, "/", MYLEXER_DIV);
	llex_add_token(&lex, "*", MYLEXER_MULT);
	llex_add_token_regex(&lex, "\\s+", MYLEXER_WHITESPACE);
	llex_add_token_regex_callback(&lex, "\\d+", number_callback);
	
	while ((token_id = llex_tokenizer(&lex)) > 0) {
		printf("Token id: %d - State: %d - '%.*s'\n",
			token_id, 
			lex.current_state,
			lex.current_len,
			lex.current_token);
	}
	if (token_id == -1) {
		printf("Unknown string `%s'\n", lex.current_token);
	}
	
	llex_cleanup(&lex);
	
	return 0;
}
```

Outputs:

```
Token id: 9 - State: 0 - '1'
Token id: 8 - State: 0 - ' '
Token id: 2 - State: 0 - '-'
Token id: 8 - State: 0 - ' '
Token id: 9 - State: 0 - '2'
Token id: 8 - State: 0 - ' '
Token id: 1 - State: 0 - '+'
Token id: 8 - State: 0 - ' '
Token id: 9 - State: 0 - '3'
Token id: 8 - State: 0 - ' '
Token id: 3 - State: 0 - '/'
Token id: 8 - State: 0 - ' '
Token id: 9 - State: 0 - '4'
Token id: 8 - State: 0 - ' '
Token id: 6 - State: 1 - '/*'
Token id: 10 - State: 1 - ' ignored str '
Token id: 7 - State: 0 - '*/'
```
