---
title: "Экранирование симоволов в Groovy (Jenkins pipeline)"
date: "2020-06-15"
categories:
  - "devops"
tags:
  - "jenkins pipeline"
  - "groovy escape character"
  - "groovy экранирование символов"
  - "groovy экранирование кавычек"
  - "groovy экранирование"
---
В задании **Jenkins** есть такое определение:

```groovy
def pR = sh(script: "cd $IT; PLAN=\$(terragrunt plan --terragrunt-source-update | landscape);
                    echo "$PLAN"; CHANGES=$(echo "$PLAN" | tail -2); echo $CHANGES")
```
Возникает ошибка когда идет попытка выполнить `echo "$PLAN"`:

```bash
solution: either escape a literal dollar sign "\$5" or bracket the value
expression "${5}" @ line 34, column 148.
ce-update | landscape); echo "$PLAN"; CH
```
<!--more-->

Для понимания почему возникает ошибка нужно рассмотреть механику работы декларативного конвеера (declarative pipeline) в **Jenkins**. Декларативный конвеер в **Jenkins** это скрипт на **Groovy**. **Groovy**, по сути, это командный интерпретатор аналогичный **Bash**. Принципы работы схожи с другими интерпретаторами.

После того как **Jenkins** начинает выполнять задание, **Groovy** интерпретатор начинает последовательно исполнять команды **Groovy** cкрипта. Когда доходит до места исполнения команд **Shell**, происходит запуск нового экземпляра **Shell** (подпроцесса) в котором происходит запуск команд **Shell**.
К этому моменту **Groovy** интерпретатор должен уже преобразовать все переменные которые ему встретились. Так как синтаксис определения перменных у **Bash** и **Groovy** одинковый, то он преобразовывывает в том числе переменные которые относятся к **Bash** командам. А так как этих переменных в его среде окружения нет, то возникает ошибка выше.

Таким образом, для строк с двойными кавычками, **Groovy** процессор будет преобразовывать строку первым. И только после преобразования переменных **Groovy**, запустится отдельный подпроцесс, где преобразованием переменных займется уже **Bash**.

`IT, PLAN, CHANGES` переменные среды исполнения (runtime variable) **Bash**, а не переменные среды исполнения **Groovy**. Во время преобразования переменных `IT, PLAN, CHANGES` **Groovy** не может найти соответствующие переменные из набора переменных.

Поэтому, что бы **Groovy** не смог их интерпретировать, нужно экранировать все **"$"**, если используется двойные кавычки, как в данном случае:

```groovy
def pR = sh(script: "cd \$IT; PLAN=\$(terragrunt plan --terragrunt-source-update | landscape);
                    echo \$PLAN; CHANGES=\$(echo \$PLAN | tail -2); echo \$CHANGES")

```

Или использовать одиночные кавычки, которые не используют преобразования:

```groovy
def pR = sh(script: 'cd $IT; PLAN=$(terragrunt plan --terragrunt-source-update | landscape);
                    echo $PLAN; CHANGES=$(echo $PLAN | tail -2); echo $CHANGES')

```

Использованная литература - [Stackoverflow One](https://stackoverflow.com/questions/51659231/groovy-escape-double-quoted/51662852), [Stackoverflow Second](https://stackoverflow.com/questions/59171237/jenkins-script-console-can-i-use-jenkins-pipeline-dsl-in-script-console)
