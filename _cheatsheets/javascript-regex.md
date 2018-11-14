---
layout: cheatsheet
title:  "Javascript Regex Cheat Sheet"
categories: emoji
tags: [cheetsheet emoji]
---
<div class="cheatsheet">
  <div class="command-container">
    <div class="grid-block">
      <div class="grid-lg-1-2">
        <h2>Basics</h2>
        <ul>
          <li><kbd>.</kbd> - Any character except newline</li>
          <li><kbd>a</kbd> - The character a</li>
          <li><kbd>ab</kbd> - The string ab</li>
          <li><kbd>a|b</kbd> - a or b</li>
          <li><kbd>a\*</kbd> - 0 or more a's</li>
          <li><kbd>\</kbd> - Escapes a special character</li>
        </ul>
        <h2>Quantifiers</h2>
        <ul>
          <li><kbd>*</kbd> - 0 or more</li>
          <li><kbd>+</kbd> - 1 or more</li>
          <li><kbd>?</kbd> - 0 or 1</li>
          <li><kbd>{2}</kbd> - Exactly 2</li>
          <li><kbd>{2,5}</kbd> - Between 2 and 5</li>
          <li><kbd>{2,5}</kbd> - Between 2 and 5</li>
        </ul>
        <h2>Groups</h2>
        <ul>
          <li><kbd>(..)</kbd> - Capturing group</li>
          <li><kbd>(?:..)</kbd> - Non-capturing group</li>
          <li><kbd>\Y</kbd> - Match the Y'th captured group</li>
        </ul>
        <h2>Character Classes</h2>
        <ul>
          <li><kbd>[ab-d]</kbd> - One character of a, b, c, d</li>
          <li><kbd>[^ab-d]</kbd> - One character except a, b, c, d</li>
          <li><kbd>[\b]</kbd> - Backspace character</li>
          <li><kbd>\d</kbd> - One digit</li>
          <li><kbd>\D</kbd> - One non-digit</li>
          <li><kbd>\s</kbd> - One whitespace</li>
          <li><kbd>\S</kbd> - One non-whitespace</li>
          <li><kbd>\w</kbd> - One word character</li>
          <li><kbd>\w</kbd> - One non-word character</li>
        </ul>
      </div>
      <div class="grid-lg-1-2">
        <h2>Assertions</h2>
        <ul>
          <li><kbd>^</kbd> - Start of string</li>
          <li><kbd>$</kbd> - End of string</li>
          <li><kbd>\b</kbd> - Word boundary</li>
          <li><kbd>\B</kbd> - Non-word boundary</li>
          <li><kbd>(?=..)</kbd> - Positive look ahead</li>
          <li><kbd>(?!..)</kbd> - Negative look ahead</li>
        </ul>
        <h2>Flags</h2>
        <ul>
          <li><kbd>g</kbd> - Global match</li>
          <li><kbd>i</kbd> - Ignore case</li>
          <li><kbd>m</kbd> - ^ and $ match start and end of line</li>
        </ul>
        <h2>Special Characters</h2>
        <ul>
          <li><kbd>\n</kbd> - New line</li>
          <li><kbd>\r</kbd> - Carriage return</li>
          <li><kbd>\t</kbd> - Tab</li>
          <li><kbd>\0</kbd> - Null Character</li>
          <li><kbd>\YYY</kbd> - Octal character YYY</li>
          <li><kbd>\xYY</kbd> - Hexadecimal character YY</li>
          <li><kbd>\uYYYY</kbd> - Hexadecimal character YYYY</li>
          <li><kbd>\cY</kbd> - Control character Y</li>
        </ul>
        <h2>Replacement</h2>
        <ul>
          <li><kbd>$$</kbd> - Insert $</li>
          <li><kbd>$&</kbd> - Insert entire match</li>
          <li><kbd>$`</kbd> - Insert preceding string</li>
          <li><kbd>$'</kbd> - Insert following string</li>
          <li><kbd>$Y</kbd> - Insert Y'th captured group</li>
        </ul>
      </div>
    </div>
  </div>
</div>
