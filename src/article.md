# Hello World

This is a proof-of-concept, work in progress version of my portfolio. It is generated almost completely from Markdown files contained in an S3 bucket. This makes maintenance and adding to the blog very easy in the long term. It is also auto-deployed via GitHub Actions!

* Markdown

In line `code`

Hello World, auto deployed from GitHub Actions!

> A block quote with ~strikethrough~ and a URL: https://reactjs.org.

* Lists
* [ ] todo
* [x] done

<Code language="python">
def main():
    print("Hello")
</Code>

<Code language="java">
public static void main(String[] args){
    ArrayList\<Integer\> ints;
}
</Code>

<Code language="javascript">
const helloWorld = () => {
    console.log("Hello World")
}
</Code>

<Code language="java">
name: Publish

on:
    push:
        branches: main
</Code>

Hello!

Also this is not cached.