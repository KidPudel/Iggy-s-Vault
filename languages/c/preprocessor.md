Preprocessor is the *program*, that processes the source code as a plain text, before it gets compiled by the compiler.
It handles [[directives in C]]

Main tasks:
	1. File inclusion: Preprocessor handles `#include`, which includes other source code or headers to the file. This makes code modular, meaning, it allows reuse of code.
	2. Macro Expansion: Preprocessor expands [[macros]] with `#define` directive, which can be used to define constants, code snippets, and text substitution (replace directive with a code block)
	3. Conditional Compilation: Preprocessor evaluates conditional directives like `#if`, `#elif`, `#else`, `#endif`, this allows the inclusion or exclusion of specific code blocks, which can be useful for handling different platform specific configurations.
	4. Line control: `#line` for debugging purposes or when working with generated code.
	5. Error handling: Preprocessor can generate an error message using `#error` directive, that is useful when catching errors or enforcing specific requirements.

> Note: Modern C programming practices often favour alternative techniques, such as [[inline functions]] or [[constexpr]], over complex macro definitions, whenever possible.