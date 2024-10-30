Months ago when coming up with the idea to start this blog alongside my portfolio website, I came up with the idea that I wanted my blog posts to be written entirely in Markdown, which will be converted to JSX automatically and displayed in my React frontend. 

The motivation for wanting this Markdown to JSX conversion was rooted mostly in curiosity and a chance to experiment rather than providing added convenience and practicality in the blog writing process. Although it might save me some time per blog, it was quite tedious to get this plugin and site setup in the way I wanted. This demotivated me at first, resulting in me stepping away from this project for some time, but with this guide I aim to alleviate the pain in getting this plugin setup and working for your needs.

After some research and experimenting with different tools, I found the `markdown-to-jsx` plugin (see plugin page [here](https://www.npmjs.com/package/markdown-to-jsx)), which seemed like the best documented and most compatible solution for my use case.

## Basic Usage

First, you need to install the plugin which is as simple as `npm i markdown-to-jsx`. The following React code snippet is a simple start to rendering Markdown as JSX components:

<Code language='jsx'>
import Markdown from "markdown-to-jsx";
const myComponent = () => {
    return (
        \<Markdown># Hello World!</Markdown>
    )
};
export default Code;
</Code>

This very basic example has a very major flaw: JSX does not preserve newlines natively in multiline text, so writing multiple lines of Markdown directly into JSX would *not* be a good idea! The code can be modified as follows to generate JSX from a `.md` Markdown file:

<Code language='jsx'>
import Markdown from "markdown-to-jsx";
import { useEffect, useState } from 'react';
// Path to your markdown file
// If this path is going to be a local file, it MUST be in the 'public/' directory or there will be a 404 error
const contentPath = 'path/to/markdown/file.md'
const myComponent = () => {
    // Create state variable to load markdown file content into
    const [markdownContent, setMarkdownContent] = useState("");
    // Run on component load
    useEffect(() => {
        fetch(contentPath)
        .then((response) => response.text())
        .then((data) => setMarkdownContent(data))
        .catch((error) => console.log(error));
    }, [contentPath]);
    return (
        \<Markdown>{markdownContent}</Markdown>
    )
};
export default Code;
</Code>

## Advanced Usage

The previous code snippet works for generating JSX from Markdown, but some components may not render as you would expect. These can be overwritten and replaced with a custom JSX component. Furthermore, you can define custom HTML tags to be parsed in your Markdown which can be rendered as custom Components. To override which component gets rendered for certain markdown files, the following code can be used:

<Code language='html'>
\<Markdown options={{overrides: {markdownStyle: {component: myComponent}}}}>
{markdownContent}
\</Markdown>
</Code>

> In this example, replace `markdownStyle` with a style you want to override, and `myComponent` with the custom component you have created. The following examples will be showing how to do this for basic Markdown syntax you would expect to work natively, but is not provided by the plugin.

### Blockquote

The first Markdown syntax from my everyday markdown usage that I expected to work natively with the plugin was using `>` at the beginning of a line to render a blockquote.

> This is a blockquote for those who are confused.

Overriding the `blockquote` style with my custom component `Blockquote.jsx` was a simple fix to this issue.

- `Markdown` component override option:
<Code language='html'>
\<Markdown options={{overrides: {blockquote: {component: Blockquote}}}}>
{markdownContent}
\</Markdown>
</Code>

- `Blockquote.jsx`:
<Code language='jsx'>
const Blockquote = ({ children }) => {
    return (
        \<div className="blockquote">
            {children}
        </div>
    )
}
export default Blockquote
</Code>

- CSS for styling:
<Code language='css'>
.blockquote {
    color: #7d8590;
    border-left-style: solid;
    border-left-width: 3px;
    border-left-color: #7d8590;
    padding-left: 15px;
    font-family: -apple-system,BlinkMacSystemFont,"Segoe UI","Noto Sans",Helvetica,Arial,sans-serif,"Apple Color Emoji","Segoe UI Emoji";
    line-height: 1.5;
}  
</Code>

> Disclaimer: I copied this color scheme from Github. Replace the colors as you see fit.

### Inline Code

Another syntax that I use very commonly is for specifying inline code, especially when referring to variables/code in text - e.g. `x`, `npm i`, `sqrt(x)`.

This time we have to override the `code` component, I created a custom component called `InlineCode` for this:

- `Markdown` component override option:
<Code language='html'>
\<Markdown options={{overrides: {code: {component: InlineCode}}}}>
{markdownContent}
\</Markdown>
</Code>

- `InlineCode.jsx`
<Code language='jsx'>
const InlineCode = ({ children }) => {
    return (
        \<span className="inline-code">
            {children}
        </span>
    )
}
export default InlineCode
</Code>

- CSS for styling:
<Code language='css'>
.inline-code {
    background-color: rgba(110, 118, 129, 0.4);
    padding: 3px 5px;
    border-radius: 6px;
    font-size: small;
    color: #e6edf3;
    font-family: ui-monospace,SFMono-Regular,SF Mono,Menlo,Consolas,Liberation Mono,monospace;
    cursor: pointer;
}
</Code>

### Strikethrough

Another syntax I find myself using in Markdown is `~<text>~` for having a line through the text (like <Line>this</Line>). However that syntax doesn't work. I also couldn't find an existing style in `Markdown` to override. Therefore, this is our first example of creating a custom HTML tag to be parsed in the markdown to be rendered as a custom component we will create. I chose to make my HTML tag `<Line>{text}</Line>` for this, but you can name it whatever you want. I named my custom component `Strikethrough.jsx`.

- `Markdown` component override option:
<Code language='html'>
\<Markdown options={{overrides: {Line: {component: Strikethrough}}}}>
{markdownContent}
\</Markdown>
</Code>

> Remember to replace `Line` with whatever you want your custom HTML tag to be named!

- `Strikethrough.jsx`
<Code language='jsx'>
const Strikethrough = ({ children }) => {
    return (
        \<span className="strikethrough">
            {children}
        </span>
    )
}
export default Strikethrough
</Code>

- CSS for styling
<Code language='css'>
.strikethrough {
    text-decoration: line-through;
}
</Code>

### Code Blocks

One of the most important features of Markdown syntax that I wanted to leverage for my blog was block code. Using ``` as a delimeter you can enter code and it will display as a code block, and you can even specify a language for syntax highlighting. Both these features were very important to me and didn't come natively with the plugin. This took some hacking to get working, and I will share my results with you.

First, I defined a custom `<Code>` html tag since I couldn't figure out if the plugin had a native style for the code block like it did for `blockquote` and `code` (remember that this was inline code, not a code block). An amazing feature of the `Markdown` component is that you can have options from the custom HTML tag be passed into your JSX component as props. In this case, I wanted to retain the language specific syntax highlighting from Markdown, and I wanted to achieve this by passing in a language name to my html tag like `<Code language='java'>/*code*/</Code>`. By adding `language` as a prop to my `Code.jsx` component, the `Markdown` component knows to pass in any value for `language` found in my Markdown `<Code>` tag as a prop.

Now that we know what language the given code is in, we still need some engine to parse and highlight the syntax. I found the `react-syntax-highlighter` library perfect for this. The `SyntaxHighlighter` component can be used for our needs, and the plugin offers multiple color schemes:

<Code language='jsx'>
import { PrismLight as SyntaxHighlighter } from 'react-syntax-highlighter';
import { oneDark } from 'react-syntax-highlighter/dist/esm/styles/prism';
// Use the following two lines to register each language you would like to support syntax highlighting for
import python from 'react-syntax-highlighter/dist/esm/languages/prism/python';
SyntaxHighlighter.registerLanguage('python', python);
const myComponent = () => {
    return (
        \<SyntaxHighlighter
            language='python'
            style={oneDark}
        >
            // Code
        </SyntaxHighlighter>
    )
}
</Code>

The code snippet shown above uses the `oneDark` color scheme offered in the plugin, but check the [docs](https://www.npmjs.com/package/react-syntax-highlighter) for more info on other color schemes. The `customStyle` prop can also be passed in to define custom styling to the component. For my application I wanted rounded borders on the code block so I have passed in `customStyle={{borderRadius: "10px"}}`.

Another nice to have feature which I always appreciate when reading other blog posts or tutorials is the ability to copy the entire code displayed in a code block by clicking a copy to clipboard icon in the corner. The `react-copy-to-clipboard` library worked perfectly for this use case, along with some `FontAwesome` icons.

Putting it all together involves passing in override options in `Markdown`, creating our custom `Code` component, and some styling:

- `Markdown` component override options:
<Code language='html'>
\<Markdown options={{overrides: {Code: {component: Code}}}}>
{markdownContent}
\</Markdown>
</Code>

- `Code.jsx`:
<Code language='jsx'>
// For copying to clipboard
import { useEffect, useState } from 'react';
import CopyToClipboard from 'react-copy-to-clipboard';
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome';
import { faClone, faCheck } from '@fortawesome/free-solid-svg-icons';
// Syntax Highlighting
import { PrismLight as SyntaxHighlighter } from 'react-syntax-highlighter';
import { oneDark } from 'react-syntax-highlighter/dist/esm/styles/prism';
// Repeat these lines for each language you want to support
import python from 'react-syntax-highlighter/dist/esm/languages/prism/python';
SyntaxHighlighter.registerLanguage('python', python)
const Code = ({ children, language }) => {
    // State variables for giving user visual feedback the copy to clipboard was succesful
    const [copied, setCopied] = useState(false);
    useEffect(() => {
        const timer = setTimeout(() => {
            setCopied(false)
        }, 1000)
        return () => clearTimeout(timer)
    }, [copied])
    return (
        \<div className="code">
            \<CopyToClipboard text={children} onCopy={() => setCopied(true)}>
                \<button className='icon copy-icon'>
                    {copied ? \<FontAwesomeIcon icon={faCheck} /> : \<FontAwesomeIcon icon={faClone} />}
                </button>
            </CopyToClipboard>
            \<SyntaxHighlighter
                language={language}
                style={oneDark}
                customStyle={{borderRadius: "10px"}}
            >
                {children}
            </SyntaxHighlighter>
        </div>
    )
}
export default Code
</Code>

- CSS for styling:
<Code language='css'>
.code {
  position: relative;
}
.code .copy-icon {
  position: absolute;
  top: 1rem;
  right: 1rem;
  z-index: 5;
}
.copy-icon {
  color: #e6edf3;
  cursor: pointer;
}
</Code>

In my final implementation I have the following code for setting up the syntax highlighting for all the code I need:

<Code language='javascript'>
// Imports
import { PrismLight as SyntaxHighlighter } from 'react-syntax-highlighter';
import { oneDark } from 'react-syntax-highlighter/dist/esm/styles/prism';
import python from 'react-syntax-highlighter/dist/esm/languages/prism/python';
import java from 'react-syntax-highlighter/dist/esm/languages/prism/java';
import yaml from 'react-syntax-highlighter/dist/esm/languages/prism/yaml';
import json from 'react-syntax-highlighter/dist/esm/languages/prism/json';
import javascript from 'react-syntax-highlighter/dist/esm/languages/prism/javascript';
import xml from 'react-syntax-highlighter/dist/esm/languages/prism/xml-doc';
import css from 'react-syntax-highlighter/dist/esm/languages/prism/css';
import jsx from 'react-syntax-highlighter/dist/esm/languages/prism/jsx';
// Registering language names
SyntaxHighlighter.registerLanguage('python', python);
SyntaxHighlighter.registerLanguage('java', java);
SyntaxHighlighter.registerLanguage('yaml', yaml);
SyntaxHighlighter.registerLanguage('json', json);
SyntaxHighlighter.registerLanguage('javascript', javascript);
SyntaxHighlighter.registerLanguage('html', xml);
SyntaxHighlighter.registerLanguage('css', css);
SyntaxHighlighter.registerLanguage('jsx', jsx);
</Code>

> Check the [Github repo](https://github.com/EmilSoleymani/portfolio) for this project for the most up to date code.

For highlighting the React code given in this blog, I used the `jsx` language. Also it's worth noting that I couldn't find `html` highlighting, so I used the `xml` highlighting instead.