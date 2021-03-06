# ParsePage
ParsePage is a template parser engine, good for web development

## Languages supported
- [PHP](php)
- [.NET](dot.net)
- [Javascript](js) (limited features for now)
- Perl (deprecated)

## Overview

When you create a website you have to generate html code for the client's browser. There are many ways how to do that, but one of the best practices is to separate _code_, _data_ and _design_. It works as: _data_ stored in some database, _code_ read it, combine with _design_ and output to browser. Something like that implemented in modern MVC web frameworks.

Separating _design_ from _code_ and _data_ can be done using _templates_. Therefore each html page generated for browser should be created with one or more templates.

Websites contains many pages, but usually most of pages have same layout (i.e. top nav, aside nav, main content area, footer).

**Core idea of ParsePage templates** is to have main page template and then build final page from sub-templates stored in directory specific for particular page. This is a bit different approach from what other template engines offer.

Templates contains special tags `<~tag>`, which replaced by actual data passed to ParsePage engine or by content of other templates (i.e. templates may include other templates).

TODO - add image here how templates works

## Sample

### Sample main template:

_/layout.html_
```html
<!DOCTYPE html>
<html lang="en"><head></head>
<body>
  <h1><~title></h1>
  <~main>
  <~/common/footer>
</body>
</html>
```

### Sub-templates for test page:

_/test/title.html_
```html
Test page header
```

_/test/main.html_
```html
<p><~data></p>
<p><~more_data></p>
```

### Common Sub-templates:

_/common/footer.html_
```html
<footer>site footer<footer>
```

### Sample code.
This will load `layout.html` layout and insert/parse sub-templates from `/test` directory, then output to browser;
```php
$ps=array(
  'data' => 'Hello World!',
  'more_data' => 12345,
);
parse_page('/test', '/layout.html', $ps);
```

this will output:
```html
<!DOCTYPE html>
<html lang="en"><head></head>
<body>
  <h1>Test page header</h1>
  <p>Hello World!</p>
  <p>12345</p>
  <footer>site footer<footer>
</body>
</html>
```

## Documentation

### /template directory structure

All templates should be placed under single directory accessible by website code, we name it `/template` (relative to website root).
**Sample** structure:
```
layout.html  - main template for website pages
layout_print.html - main template for printed pages
layout_simple.html - main template for website pages with some simpler layout
common/ - directory with common sub-templates used by multiple layouts
home/ - directory with sub-templates for website Home page
contact/ - directory with sub-templates for website Contact page
```

### Code

Generally ParsePage called as following (depending on language syntax will differ):

`parse_page(BASE_DIR, PAGE_TPL_FILE, PS, [OUTPUT_FILENAME])`

where:
- `BASE_DIR` - directory (relative to /template) with sub-templates for current page
- `PAGE_TPL_FILE` - path (relative to /template) to main page template/layout used for current page
- `PS` - "parse strings" - associative array (hashtable) with variable names/values to be replaced in templates
- `OUTPUT_FILENAME` - optional, output file name, if defined ParsePage will write output to this file, instead of output to browser

### Tag syntax

Tags in templates should be in `<~tag_name [optional attributes]>` format.

`tag_name` could be:

- **name of the variable** passed to ParsePage engine. In this case ParsePage replace tag with variable value (optionally processed according attributes).
  - if variable is an array/hashtable squares can be used to get particular keys/index - `<~var[aaa][0][bbb]>`
  - if no variable with such name passed ParsePage will look for sub-template file with such name (see below)
- **path to sub-template**. In this case ParsePage will look for sub-template file, parse it recursively and replace tag with parsed content
  - if no file found tag repalced with empty string
  - path can be:
    - `tag_name` - will look for `tag_name.html` in `BASE_DIR` (i.e. default sub-template extension is `html`)
    - `tag_name.ext` - will look for `tag_name.ext` in `BASE_DIR`
    - `./tag_name` - (relative path) will look for `tag_name.html` in directory relative to currently parsed template file (see samples)
    - `/subdir/tag_name` - (absolute path) will look for `/subdir/tag_name.html` relative to `/template` directory

Note, CSRF shield integrated - all vars escaped, if var shouldn't be escaped use `noescape` attr: `<~raw_variable noescape>`

<table>
  <thead><tr><th>PS</th><th>Template</th><th>Output</th></tr></thead>
  <tbody>
    <tr>
      <td>
<pre lang="php">
'username' => 'John',
'user' => array(
  'id' => 123,
  'status' => 'Active'
),
'orders' => array(
  array(
    'no'=>1,
    'item'=>'Pen'
  ),
  array(
    'no'=>2,
    'item'=>'Pencil'
  )
),
'csrf' => '&lt;b&gt;test&lt;/b&gt;',
</pre>
      </td>
      <td>
<pre>
User: <~username><br>
Status: <~user[status]><br>
First item: <~orders[0][item]><br>
Test: <~csrf>
</pre>
      </td>
      <td>
User: John<br>
Status: Active<br>
First item: Pen<br>
Test: &lt;b&gt;test&lt;/b&gt;
      </td>
    </tr>
  </tbody>
</table>


### Supported attributes

- `var` - tag is variable, no fileseek necessary even if no such variable passed to ParsePage: `<~tag var>`
- `ifXX` - if confitions, if false - tag replaced with empty string
  - `<~tag ifeq="var" value="XXX">` - tag/template will be parsed only if var=XXX
  - `<~tag ifne="var" value="XXX">` - tag/template will be parsed only if var!=XXX
  - `<~tag ifgt="var" value="XXX">` - tag/template will be parsed only if var>XXX
  - `<~tag ifge="var" value="XXX">` - tag/template will be parsed only if var>=XXX
  - `<~tag iflt="var" value="XXX">` - tag/template will be parsed only if var<XXX
  - `<~tag ifle="var" value="XXX">` - tag/template will be parsed only if var<=XXX
- `vvalue` - used with `ifXX` conditions to indicate that value to compare should be read from PS variable
  - `<~tag ifeq="var" vvalue="var2">` - tag parsed if PS['var']==PS['var2']
- `if`
    - `<~tag if="var">` - tag parsed if var is evaluated as TRUE, equivalent to `if ($var)`
- `unless`
    - `<~tag unless="var">` - tag parsed if var is evaluated as TRUE, equivalent to `if (!$var)`
    - TRUE values:
      - non-empty string, but not equal to '0'!
      - 1 or other non-zero number
      - true (boolean)
    - FALSE values:
      - '0'
      - 0
      - false (boolean)
      - ''
      - unset/undefined variable

<table>
  <thead><tr><th>PS</th><th>Template</th><th>Output</th></tr></thead>
  <tbody>
    <tr>
      <td>
<pre lang="php">
'username' => 'John',
'is_logged' => true,
'is_admin' => false,
</pre>
      </td>
      <td>
<pre>
User: <~username if="is_logged">
<~adm if="is_admin" inline>(admin)< /~adm>
<~usr unless="is_admin" inline>(user)< /~usr>
</pre>
      </td>
      <td>
User: John (user)
      </td>
    </tr>
  </tbody>
</table>

- `repeat` - this tag is repeat content (PS should contain reference to array of hashtables),
  - `<~tag repeat inline>repeated sub-template content</~tag>` - inline sample
  - inside repeat sub-template supported repeat vars:
    - repeat.first (0-not first, 1-first)
    - repeat.last  (0-not last, 1-last)
    - repeat.total (total number of items)
    - repeat.index  (0-based)
    - repeat.iteration (1-based)

<table>
  <thead>
    <tr><th>PS</th><th>Template</th><th>Output</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>
<pre lang="php">
'rows' => db_array('select fname from users')
</pre>
      </td>
      <td>
<pre>
<~rows repeat inline>
  <~fname>
  <~comma unless="repeat.last">,< /~comma>
< /~rows>
</pre>
      </td>
      <td>
        John, Emma, Walter
      </td>
    </tr>
    <tr>
      <td>
<pre lang="php">
'rows' => db_array('select fname from users')
</pre>
      </td>
      <td>

```
  <~rows repeat>
```

rows.html template file:
```
  <~fname>
  <~comma unless="repeat.last">,</~comma>
```

</td>
      <td>
        John, Emma, Walter
      </td>
    </tr>
  </tbody>
</table>

- `sub` - this tag tells parser to use sub-hashtable for parse sub-template (PS variable should contain reference to hashtable)
- `inline` - this tag tells parser that sub-template is not in file - it's between <~tag>...</~tag> , useful in combination with 'repeat' and 'if'. In case of 'if' tag name doesn't really matter and not need to exists in PS. **Note** if you have 2 or more inline tags with exactly the same name and attributes in same template file, content of all of them will be replaced by inline sub-template of the first tag. In the example below it will output "sold!" in both cases (i.e. not "already sold"):
  - workaround for this - use different labels. For below example just add "2" like `<~somelabel2...</~somelabel2>`
  - **Important - use `</~tag>` for closing tags, i.e. backslash then tilde**
```
     <~somelabel inline if="is_sold">
       <div>sold!</div>
     </~somelabel>
     ...other content in same file...
     <~somelabel inline if="is_sold">
       <div>already sold</div>
     </~somelabel>
```
  

- `parent` - this tag need to be read from paren't PS var, not in current PS hashtable (usually used inside `repeat` sub-templates), example:
```
     <~rows repeat inline>
       <~var_in_rows> <~var_in_parent_ps parent>      
     </~rows>
```

- `select="var"` - this tag tells parser to load file with tag name and use it as "value|display" for `<select>` html tag, example:
```
     <select name="item[fruit]">
       <option value=""> - select a fruit -</option>
       <~./fruits.sel select="fruit">
     </select>
     
     and file fruits.sel contains:
     10|Apple
     20|Cherry
     30|Grape
     
     if, for example, PS contains 'fruit'=>10, then Apple will be selected by default.
```

- `radio="var" name="YYY" [delim="ZZZ"]` - this tag tell parser to load file and use it as value|display for <input type=radio> html tags, example: `<~fradio.sel radio="fradio" name="item[fradio]" delim="&nbsp;">`
- `selvalue="var"` - display value (fetched from the tag name file) for the var (example: to display 'select' and 'radio' values in List view), example: `<~fcombo.sel selvalue="fcombo">`
  - **Note**, it doesn't work for direct values, you can't write `<~fcombo.sel selvalue="10">`. Instead put 10 into PS var and use var name.

<table>
  <thead>
    <tr><th>PS</th><th>Template</th><th>Output</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>
<pre lang="php">
'fruit' => 20
</pre>
      </td>
      <td>
<pre>
<~fruits.sel selvalue="fruit">

fruits.sel file:
10|Apple
20|Cherry
30|Grape
</pre>
      </td>
      <td>
Cherry
      </td>
    </tr>
  </tbody>
</table>

- `htmlescape` - replace special symbols by their html equivalents (such as <>,",'). This attribute is applied automatically to all tags by default.
- `noescape` - will not apply htmlescape to tag value
- `date` - will format tag value as date, format depends on language: [PHP](http://php.net/manual/en/function.date.php), [ASP.NET](https://msdn.microsoft.com/en-us/library/8kb3ddd4(v=vs.80).aspx)
  - `<~tag date>` or `<~tag date="d M Y H:i">`
- `truncate` - truncates a variable to a character length (default 80), optionally - append trchar if truncated, truncate at word boundary (trword), truncate at end or in the middle (trend)
  - `<~tag truncate="80" trchar="..." trword="1" trend="1">` - default values
- `strip_tags` - remove any html tags from value
- `trim` - remove leading and trailing space from value
- `nl2br` - convert newline chars to `<br>`
- `count` - ouput count of elements in value instead of value (for arrays only)
- `lower` - convert value to lowercase
- `upper` - convert value to uppercase
- `default` - if value is empty, ouput default value instead
  - `<~tag default="none">`
- `urlencode` - encode value for URLs
- `var2js` - ouput variable as JSON string
- `markdown` - convert markdown text to html using appropriate library for the language (ASP.NET - CommonMark.NET). Note: may wrap tag with <p>


- `<~session[var]>` - this tag is a SESSION variable, not in PS hashtable (for PHP it's $_SESSION, for ASP.NET it's current context's HttpSessionState object)
- `<~global[var]>` - this tag is a global var, not in PS hashtable (for PHP it's $_GLOBALS, for ASP.NET it depends on framework global storage implementation)

### Commenting tags

- to comment a `<~sometag>` so it won't be displayed just prepend some char to it's name, like '#', for example: `<~#sometag>` (it's just a simple workaround assuming you don't have '#sometag' var in PS and there are no such filename in your template dir).
- to comment an inline tag like `<~rows repeat inline>...</~rows>`, just wrap it into other inline tag with false condition, for example:
```
  <~hide if="#" inline>
    <~rows repeat inline>...</~rows>
  </~hide>
```

### Multi-language support

PHP and ASP.NET implementations support multi-lanugage templates.
Text that need to be replaced according to user's display language should be placed in templates between backtick characters.
Parser then look for replacements in `/template/lang/$lang.txt` files, where `$lang` is "en", "es", "ua", for example.
You need to create these files and fill with replacements in this format:
```
  english string === string in another language
```
Default lanugage is English, so en.txt doesn't need be created.

For example, you have a template:
```
  `Hello` <~username>
```

And `/template/lang/es.txt` file for Spanish translations contains line:
```
    Hello === Hola
```

If user's language is English, parser will output `Hello John`.
And if user's language will be Spanish, parser will output `Hola John`.

