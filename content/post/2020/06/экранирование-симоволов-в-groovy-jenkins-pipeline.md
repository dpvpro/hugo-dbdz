---
title: "Экранирование симоволов в Groovy (Jenkins pipeline)"
date: "2020-06-15"
categories:
  - "soft"
tags: 
  - "jenkins pipeline"
  - "groovy escape character"
  - "groovy экранирование символов"
  - "groovy экранирование кавычек"
  - "groovy экранирование"
---
В задании Jenkins есть такое определение:

```groovy
def pR = sh(script: "cd $it; PLAN=\$(terragrunt plan --terragrunt-source-update | landscape);
                     echo "$PLAN"; CHANGES=$(echo "$PLAN" | tail -2); echo $CHANGES")
```
Возникает ошибка когда идет попытка выполнить `echo "$PLAN"`:

```bash
solution: either escape a literal dollar sign "\$5" or bracket the value 
expression "${5}" @ line 34, column 148.
ce-update | landscape); echo "$PLAN"; CH
```
<!--more-->

Происходит это из-за особенностей интерпретации кода в **Jenkins**. Для строк с двойными кавычками, **Groovy** процессор будет преобразовывать строку первым. И только после преобразования переменных **Groovy**, преобразованием переменных займется **Bash**.

`it, PLAN, CHANGES` переменные среды исполнения (runtime variable) **Bash** нежели переменные среды исполнения **Groovy**. **Groovy** не может найти соответствующие переменные из стэка переменных для замены `it, PLAN, CHANGES` во время преобразования переменных.

Поэтому нужно экранировать все **"$"** если используется двойные кавычки в данном случае.

```groovy
def pR = sh(script: "cd \$it; PLAN=\$(terragrunt plan --terragrunt-source-update | landscape); 
                    echo \$PLAN; CHANGES=\$(echo \$PLAN | tail -2); echo \$CHANGES")

```

Или использовать одиночные кавычки, которые не используют преобразования:

```groovy
def pR = sh(script: 'cd $it; PLAN=$(terragrunt plan --terragrunt-source-update | landscape); 
                    echo $PLAN; CHANGES=$(echo $PLAN | tail -2); echo $CHANGES')

```

https://stackoverflow.com/questions/51659231/groovy-escape-double-quoted/51662852

https://stackoverflow.com/questions/59171237/jenkins-script-console-can-i-use-jenkins-pipeline-dsl-in-script-console
