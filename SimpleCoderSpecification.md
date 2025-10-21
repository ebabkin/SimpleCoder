SimpleCoder file format is intented for more efficient usage of LLM agents in software development.
It packs a project directory to a single file and instructs LLM on how to respond within a single file.

# 1. Format description:
1.1 The request and responce file format is a UTF-8 encoded text file.

1.2 Command lines start with `@@SC`, following by an optional action and optional parameters. 

1.3 The user creates a SimplerCoder Request file, the LLM responds with a SimplerCoder Response.

1.4 `@@SC ##` is a comment line, `@@SC /*` opens a comment block, `@@SC */` closes the block.

1.5 Path argument in commands cannot include `..`

1.6 The path argument can be enclosed in quotes if it contains spaces or special characters, or you can always use quotes for consistency.. 

1.7 For readability, there should be an empty line after each command. 

1.8 Plain text files cannot contain lines starting with `@@SC`.

1.9 Full list of commands:
```
@@SC ##
@@SC /*
@@SC */
@@SC gitstate 
@@SC tree
@@SC file [plain,  base64, redacted, end] path/filename
@@SC write [plain, base64, end] path/filename
@@SC mkdir path/dirname
@@SC delete path/filename
@@SC mv path/filename path/filename
```

# 2 SimpleCoder Request format:
The request can contain comments, the project tree, and the contents of relevant files. It’s up to the user to decide whether to include all files or only a subset of directories. 

2.1 The request can start with `@@SC gitstate` entry.

```@@SC gitstate On branch <branchname> last commit <commithash> timestamp <rfc-3339 date eg 2025-02-16T10:58:44Z+02:00>```

2.2 The request can contain a `@@SC tree` section before any file content to help the LLM understand theproject structure.
The content is the output of Linux `$tree -fi` command. 

2.3 Files in the project folder should be included as a `@@SC file ...` entry. 

2.4 For text files that the LLM agent should see, use: `@@SC file plain path/to/file/filename.ext`, 
followed by the plain text content of the file, and closed with
`@@SC file end path/to/file/filename.ext`

2.5 For binary files or files that the LLM should know exist but not read, use:
`@@SC file redacted path/to/file/filename.ext`

2.6 For binary files that the LLM should process, use:
` @@SC file base64 path/to/file/filename.ext` followed by the base64-encoded file content, and closed with `@@SC file end path/to/file/filename.ext`


# 3 SimpleCoder Response format:
The response must contain a change summary and the contents of changed project files or directories, if any.
The response must NOT include the contents of files that remain unchanged.

3.1 If `@@SC gitstate` was present in the request, the reponse must mirror it as the first entry. 

3.2 The response must include a comment block giving a LLM's own summary of the change prior actual content.

3.3 The response can contain a `@@SC tree` section before any file contents.
The content should represent the output of the Linux `$tree -fi` command for the updated project structure.

3.4 Files that should not be changed must NOT be included in the response.

3.5 Files or directories to be deleted are listed as:
`@@SC delete path/to/file/filename.ext` and `@@SC delete path/to/directory`.
Each file and directory must be listed explicitly; wildcards are not allowed.

3.6 Directories to be created are listed as: 
`@@SC mkdir path/to/directory` 

3.7 Files to be added or modified should be listed as `@@SC write ..." entries.
Each file is always provided in full and replaces the previous content.

3.8 For a plain text file, use:
`@@SC write plain path/to/file/filename.ext`
followed by the plain text content of the file,
and closed with `@@SC write end path/to/file/filename.ext`

3.9 For a binary file, use:
`@@SC write base64 path/to/file/filename.ext`
followed by the base64-encoded file content,
and closed with `@@SC write end path/to/file/filename.ext`

3.10 Files or directories that need to be renamed are listed as:
`@@SC mv path/to/oldfile.txt path/to/newfile.txt`
If a file needs to be both renamed and edited, first include the `@@SC mv` command, followed by `@@SC write` with the new name.

# 4. Request Example

```
@@SC ## This is a hello world project

@@SC gitstate On branch init last commit aca1410a8267d31ce7d26cbbbca8ffdf00510dce timestamp 2025-10-31T11:33:44Z+01:00

@@SC tree
./
./package.json
./src
./src/index.js 

@@SC file plain package.json
{
  "name": "hello-node",
  "version": "1.0.0",
  "description": "A simple Node.js Hello World program",
  "main": "src/index.js",
  "scripts": {
    "start": "node srcindex.js"
  },
  "author": "",
  "license": "MIT"
}
@@SC file end package.json

@@SC file plain src/index.js
console.log("Hello, World!");
@@SC file end src/index.js

@@SC ## End of request example
```

# 5. Response Example
If the request was to
a) change the message to "Hello, beautiful World!"
b) add a README file in plain text,
the response would look like this:

```
@@SC ## This is a response example

@@SC gitstate On branch init last commit aaa1110a1111d11ce1111cbbbca1ffdf00000bbb timestamp 2025-10-31T11:33:44Z+01:00

@@SC /*
This change would do the following:
a) change the message to "Hello, beautiful World!"
b) add readme file
@@SC */

@@SC tree
./
./package.json
./readme.txt
./src
./src/index.js 
 
@@SC write plain readme.txt
This is a Hello World in node.js

Run the program:
npm start
@@SC write end readme.txt

@@SC write plain src/index.js
console.log("Hello, beautiful World!");
@@SC write end src/index.js

@@SC ## End of response example
```