# USP WORKSHEET 03


## Q1 - grep
> grep “[mM].*[Kk]” file.txt Matches any string that starts with either m or M and ends with k or K. It can be a substring or even string with multiple words.

> grep ”<[mM].*[Kk]>” file.txt similar to above, but only when the first character is a beginning of word, and the last one is end of word. (No substring)

> grep “[mM].{2}[Kk]” file.txt Any substring with 4 characters that starts with m/M and ends with k/K.

> grep -v “[mM].{2}[Kk]” file.txt Match the inverse of previous question. -v means invert match. This is also a good time to remind them about man command man grep and search with /-v. Unix


### (a) Work out what the following commands do, then test your understanding by trying them. You will have to contrive a text file for this (or copy and paste the text at the end of this worksheet).

```
  1 grep "[mM].*[Kk]" file.txt
  2 grep "\<[mM].* [Kk]\>" file.txt
  3 grep "[mM].\{2\}[kK]" file.txt
  4 grep -v "[mM].\{2\}[Kk]" file.txt
  ```


### (b) Use grep to match lines containing:

**(i) Jefferies jeffery jeffeys**
- [Jj]effer?(y|ie)s?

**(ii) hitchen Hitchin hitching**
- [hH]itch[ei]ng?

**(iii) Heard herd Hird**
- [hH][ei]a?rd
  
**(iv) dix dicks Dickson dixon**
- [dD]i(x|ck)s?(on)?

**(v) Mcgee mcghee magee**
- [mM][ac]gh?ee


### (c) Use command substitution with grep to output the lines of the persons from file.txt who have their birthday today. Optional: man cut to find out how to output only the name.

This question requires two additional commands date and cut and piping to redirect the output. Command date is used to retrieve the current date, while command cut can be used to parse the result. Let them research these commands via command man. Since there is no sample text file, they can create their own. Let’s say the text looks like this:

`Michael:12/03/2000` `Josh:25/12/2010` `Jennifer:10/05/2018`

where each line start with first name (for simplicity) followed by a ’:’ and the date with format “dd/mm/yyyy”. The suitable command for this text file is:

`grep "$(date +%d/%m/%y)" text.txt | cut -d ":" -f 1`

This command matches the line containing the current date, and then piped to the command cut to parse based on delimiter ”:”, and we pick the first token (which is the name of the person). 

More about these commands: 
[date commands in linux](https://www.geeksforgeeks.org/date-command-linux-examples/) 
[cut commands in linux](https://www.geeksforgeeks.org/cut-command-linux-examples/)


### (d) Find the occurrences of three consecutive and identical word characters (like aaa
or bbb).

For any kind of pattern matching that depends on the previous match, we need to use backreferencing. For example, in this question we need to match three consecutive and identical characters. This requires regex to recognize the first match and then it has to repeat two more times. The simplest regex for this question is: `(.)\1\1`


### (e) Write a command to output the lines that are 50 characters in length or longer.

This question requires regex to find any string containing at least 50 characters. The simplest answer is:

`grep -E ".{50,}" text.txt`

This might lead them to ask “How about less than 50 characters?”. Keep in mind that the following regex is NOT the answer:

`grep -E ".{0,49}" text.txt`

This is because this regex will still match substring inside a very long string. The correct answer is:

`grep -E "^.{0,49}$" text.txt`

This works because it only matches short string that is between the start of the line (^) and the end of the line ($). Any string containing 50 characters or more will not get matched by this regex.



## Q2 - vi

### (a) Work out what the following commands do, then test your understanding by trying them. You will have to contrive a text file for this (or copy paste the text at the end of this worksheet)
```
1 s/ */ /g
2 s/^/ /
3 %s/^[0-9][0-9]* //
4 s/b[aeio]g/bug/g
5 s/t\([aou]\)g/h\lt/g
```

- `s/ */ /g`
Adding one space after every non-space character. For the multiple consecutive space characters, it will become a single space character. The character ’g’ (global) at the end means apply the rule on all occurences on the same line (NOT only first occurence). Character ’s’ only affect the current line where the cursor is.

- `s/^/ /`
Adding a single space character at the beginning of the line. Keep in mind that ’^’ means the beginning of the line.

- `%s/^[0-9][0-9]* //`
Matches any number at the beginning of the line (any number of digits) and remove them. One good application is to removes line number. Character “%s” means substitute on all lines in the file.

- `s/b[aeio]g/bug/g`
Replaces bag OR beg OR big OR bog all into bug.

- `s/t([aou])g/h\1t/g`
Since there is bracket () and \1, this involves backreferencing. This will substitute tag into hat, tog into hot, tug into hut.


### (b) Write a command to delete all the c++ style comments in a file

Assume we only removes any character following the double forward-slash ”//”. We can use command:

`%s/\/\/.*//`

The character ”.*” is the wildcard representing all the string after ”//” that will be removed.


### (c) Try the "Whole word only" example in the lecture notes

In order to capture whole words only, they need to use characters `"<"` and `">"`. One example from the lecture note is:

`%s/\<UNIX\>/Linux/gc`

This will replace the whole word “UNIX” into “Linux”. This applies on global scale in the file and each occurence will require confirmation from the user. (letter ’c’ for confirmation)



## Q3 - sed
Command sed is a stream editor which can be used to transform/manipulate the text on input stream. (file OR pipeline). Regex is very useful for text matching in the transformation process.
[introduction to sed](https://www.grymoire.com/Unix/Sed.html)

> Note: For interest. The following code adds line number to the start of each line
of myprog.c
> `[user@pc]$ sed = myprog.c | sed ’N;s/\n/\t/’ > myprog.c`

### (a) Write a command to output lines that are less than 50 characters in length.

You can refer to question 2(e) for the similar pattern. One possible answer is:

`cat text.txt | sed -r -n "s/^.{0,49}$/&/p"`

-r is to enable Extended Regular Expression. -n is to not print by default. The last character ’p’ is to print the matched result. The character ’&’ is to refer to the matched pattern. In this example, there is no modification, we only print the matched result.


### (b) Find out the occurrences of three consecutive and identical word characters (like aaa or bbb).

We can use backreferencing to retrieve the consecutive identical characters. Since we need to count the occurences, we can use the strategy to print each occurence on new line, and then we use command grep with -c option to count the amount of lines:

`cat text.txt | sed -r -n 's/(.)\1\1/&\n/gp' | grep -E '(.)\1\1' -c`

Notice that &\n is used to print every occurence on every new line.


### (c) What is wrong with this command? How would you correct it? [user@pc]$ sed ’s/compute/calculate/g s/computer/host/g’ file.txt

The first issue is we need to add a semi-colon ’;’ between each substitution. The second issue is the order will cause problem. If we substitute “compute” into “calculate” first, then all the words “computer” will become “calculater”. One better solution is:

`sed -r 's/computer/host/g ; s/compute/calculate/g' file.txt`

-r is to enable Extended Regular Expression.

### (d) Write a command sequence to find out the number of occurrences of the word "encryption" in a file. (Note: A word may occur more than once in a line.)

We can use the similar strategy in question (b):

`cat text.txt | sed -r -n 's/encryption/&\n/gp' | grep -E 'encryption' -c`

