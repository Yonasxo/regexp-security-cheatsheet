# regexp-fundamental-requirements

Several fundamental requirements for regexp’s were derived after observing several WAF bypass write-ups and studying tricky conditions of metacharacters. They were classified and put in the table. In the following table, middle column contain description of discovered requirement in detail; whereas right column gives a regular expression-based example, which is designed to find discovered requirement in set of rules, tuned to have minimum false positive rate.


|#| Requirement  | Regex finder  |
|---|---|---|
|1|  Regexp should avoid using ^ (alternative: \A) and $ (alternative: \Z) symbols, which are metacharacters for start and end of a string. It is possible to bypass regex by inserting any symbol in front or after regexp. | `grep -P '[^\[\\](\^|\\A|\$|\\Z)'`  |
|2| Regexp should be case-insensitive. It is possible to bypass regex using upper or lower cases in words. [Modsecurity transformation commands](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#cmdLine) (which are applied on string before regex pattern is applied) can also be included in tests to cover more regexps.  |  `grep -vP '\(\?i' | grep -v "t:lowercase" | grep -v "t:cmdLine"` |
|3| Regexp should avoid using dot “.” symbol, which means every symbol except newline (\n). It is possible to bypass regex using newline injection.  |  `grep -P '[^\\]\.\+[^*]'` |
|4|  Regexp should not be vulnerable to ReDoS. [OWASP ReDoS article](https://www.owasp.org/index.php/Regular_expression_Denial_of_Service_-_ReDoS) | 1. Find various evil patterns. Example: `grep -P '\((.*?)\)[^\\]?[\+\*]'` 2. Generate evil string using e.g. “SDL Regex Fuzzer” |
|5| Number of repetitions of set or group {} should be carefully used, as one can bypass such limitation by lowering or increasing specified numbers.  |  `grep -P '\{.*?\}'` |
|6| Best Practice from slides of Ivan Novikov: Modsecurity should avoid using t:urlDecode function (t:urlDecodeUni instead).  |  `grep -P 't:urlDecode\b'` |
|7|  Regexp should only use plus “+” metacharacter in places where it is necessary, as it means “one or more”. Alternative metacharacter star “*”, which means “zero or more” is generally preferred. |  `grep -P '\\[a-rt-vx-zA-RT-VX-Z]{1}(\+|\*)?'` This regexp tunes false positive rate to exclude wildcards \s, \S, \w, \W due to their prevalent use` |
|8| Usage of wildcards should be reasonable. \r\n characters can often be bypassed by either substitution, or by using newline alternative \v, \f and others. Wildcard \b has different meanings while using wildcard in square brackets (has meaning “backspace”) and in plain regex (has meaning “word boundary”), as classified in [RegexLib article](http://regexlib.com/CheatSheet.aspx).  | `grep -P '\\a|\[[^\]]*?\\b[^\[]*?\]|\\t|\\r|\\v|\\f|\\n'`  |
|9| Regexp should be applied to right scope of inputs: Cookies names and values, Argument names and values, Header names and values, Files argument names and content.  |  Modsecurity: `grep -oP 'SecRule(.*?)"' -n` Other WAFs: manual observation.` |
|10| Regular expression writers should be careful while using only whitespace character (%20) for separating tag attributes. Rule can be bypassed with newline character: i.e. %0d,%0a.  |  `grep -P '\[(.*)\s[^\?](.*)\]'` |
|11| Greediness of regular expressions should be considered. Highlight of this topic is well done in [Chapter 9 of Jan Goyvaert’s tutorial](https://www.princeton.edu/~mlovett/reference/Regular-Expressions.pdf). While greediness itself does not create bypasses, bad implementation of regexp Greediness can raise False Positive rate. This can cause excessive log-file flooding, forcing vulnerable rule or even whole WAF to be switched off.  |  `grep -P '[^\\]\.[\+|\*][^\?]'` |
