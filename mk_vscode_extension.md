# Make Visual Studio Code Extension

## Finished Working Example
https://github.com/smridge/vscode_rails_syntax

## Initial Setup

**Install Extension Code Generator**
```bash
npm install -g yo generator-code
```

**Generate Scaffold**
```bash
yo code
```

## The remaining Guide is how to Develop a Language Injection Grammar

**Follow Prompts**:
```
? What type of extension do you want to create?
> New Language Support

Enter the URL (http, https) or the file path of the tmLanguage grammar or press ENTER to start with a new grammar.

? URL or file to import, or none for new:

? What's the name of your extension?
> vscode_rails_syntax

? What's the identifier of your extension?
> vscode-rails-syntax

? What's the description of your extension?
> Extends Ruby Language Grammar with Rails Library

Enter the id of the language. The id is an identifier and is single, lower-case name such as 'php', 'javascript'
? Language id:
> ruby

Enter the name of the language. The name will be shown in the VS Code editor mode selector.
? Language name:
> ruby

Enter the file extensions of the language. Use commas to separate multiple entries (e.g. .ruby, .rb)
? File extensions:
> .rb

Enter the root scope name of the grammar (e.g. source.ruby)
? Scope names:
> source.ruby
```

**This Generates**:
```
create vscode-rails-syntax/syntaxes/ruby.tmLanguage.json
create vscode-rails-syntax/.vscode/launch.json
create vscode-rails-syntax/package.json
create vscode-rails-syntax/README.md
create vscode-rails-syntax/CHANGELOG.md
create vscode-rails-syntax/vsc-extension-quickstart.md
create vscode-rails-syntax/language-configuration.json
create vscode-rails-syntax/.vscodeignore
```

**Change Directory to Generated Project**
```bash
cd vscode-rails-syntax/
```

**To Help with Understanding changes made, use GIT Version Control**
```bash
vscode-rails-syntax $ git init
vscode-rails-syntax $ git add .
vscode-rails-syntax $ git commit -am "yo code generator"
```

**Open Project with Visual Studio Code**
```bash
vscode-rails-syntax $ code .
```

### Optional
- Set Indentation for Files
 - Commit Changes

## Creating a Language Injection

We are creating an `injection` to a language (not an actual new language as the scaffold generated).

Therefore we need to remove and change a few things.

<br>

**Remove** File `language-configuration.json`


### In `package.json`

**Remove**
```json
"languages": [{
  "id": "ruby",
  "aliases": ["ruby", "ruby"],
  "extensions": [".rb"],
  "configuration": "./language-configuration.json"
}],
```

**Change**
```json
"grammars": [{
  "injectTo": ["source.ruby"],
  "scopeName": "action_text",
  "path": "./syntaxes/action_text.tmLanguage.json"
}]
```

- File Name `syntaxes/ruby.tmLanguage.json` to `syntaxes/action_text.tmLanguage.json`
  - This connects `"path": "./syntaxes/action_text.tmLanguage.json"` set above.

<br>

### In `syntaxes/action_text.tmLanguage.json`

**Remove**
```json
"$schema": "https://raw.githubusercontent.com/martinring/tmlanguage/master/tmlanguage.json",
"name": "ruby",
```

**Change**
```json
"scopeName": "action_text",
```

**Add**
```json
"injectionSelector": "L:source -comment -string",
```
This tells VSCode to inject any grammars identifed into `source.ruby` excluding `comment` & `string` tokens.

<br>

### Token Mapping

We will indentify `ActionText::Attribute`.

**Change**
```json
"patterns": [
  {
    "include": "#attributes"
  }
],
```

**Change**
```json
"repository": {
  "attributes": {
    "patterns": [
      {
        "name": "support.function.attribute.action-text.rails",
        "match": "has_rich_text",
        "comment": "https://api.rubyonrails.org/classes/ActionText/Attribute.html"
      }
    ]
  }
}
```

## Testing Extension
- Open Extension Development Host by **pressing** `F5`
- Open any file ending in `.rb`
- Type `has_rich_text`
Given everything worked, syntax highlighting will be indicated.

We specifically identified `has_rich_text` as `support.function.attribute.action-text.rails`.

To validate our mapping worked correctly in VS Code:
- **press** `command` + `shift` + `p`
- **type** `scope`
- **select** `Developer: Inspect Editor Tokens and Scopes`
- **click** on recently typed `has_rich_text`

Given everything worked, hover window will indicate:

```
textmate scopes       support.function.attribute.action-text.rails
                      source.ruby
```

To Use as Extension outside of Developer Mode:
- [Create a Publisher](https://code.visualstudio.com/api/working-with-extensions/publishing-extension#create-a-publisher)
  - [UI Publisher Link](https://marketplace.visualstudio.com/manage/createpublisher)

- [Setup Azure Organization/Personal Collection](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)

- [Create Personal Access Token](https://code.visualstudio.com/api/working-with-extensions/publishing-extension#get-a-personal-access-token)

This whole process has some challenges, and may result in some frustration.

Once all the noise of the Publisher is done.

To Use Extension (without) publishing to Marketplace:

**Add**
In `package.json`
```json
"publisher": "YourPublisherName",
```

**Install** VS Code Extension Manager
```bash
npm install -g vsce
```

**Create** `.vsix` file in project root `vscode-rails-syntax`
```bash
vsce package
```

This created &nbsp;`vscode-rails-syntax-0.0.1.vsix`

**Install** Extension
```bash
code --install-extension vscode-rails-syntax-0.0.1.vsix
```

**Reload** VS Code
Check out your Extension list and see it listed!
Check out your new Syntax Highlighting!

## Publish in Marketplace

**Publish**
```bash
vsce publish
```

## Explanation

**It is crucial to understand why and how to identify and map language grammars.**

It would not be beneficial if we decided to map `has_rich_text` as an `operator`. Our Syntax Highlighting would start to look bizarre.

From [VS Code Syntax Highlight Guide](https://code.visualstudio.com/api/language-extensions/syntax-highlight-guide):

VS Code uses [TextMate grammars](https://macromates.com/manual/en/language_grammars) to break text into a list of tokens. TextMate grammars are a structured collection of [Oniguruma regular expressions](https://macromates.com/manual/en/regular_expressions) and are typically written as a plist or JSON.

Why we identified `has_rich_text` as `support.function.attribute.action-text.rails`
- `has_rich_text` is a `function` provided by the `Rails` Framework.
From [TextMate grammars](https://macromates.com/manual/en/language_grammars):
- `support` — things provided by a framework or library should be below `support`.
  - `function` — functions provided by the framework/library.
