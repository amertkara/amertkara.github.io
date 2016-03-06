---
layout: post
title: Designing a CL App with SpringBoot CommandLineRunner Interface
tags: spring-boot, java, commandlinerunner
---
SpringBoot introduces [CommandLineRunner](http://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/CommandLineRunner.html) interface which allows writing runnable spring beans that will be executed automatically in a SpringApplication. This allows writing beans that will execute some business logic once the application runs, and even the order of the runners can be determined. In this post, I hope to show how to implement the interface, order the runners and finally how to use it to implement a simple command line application that executes different runners on demand (as sub-commands).

Let's get started!

## Implementing CommandLineRunner Interface ##
Implementing the interface is simple, it only has one method that needs to be implemented; `run` method. As the name indicates, this method will be called when the Spring application loads and starts executing the runners one by one.

{% gist amertkara/41b60c81fb4023d6a851 Runner1.java %}

Above, it is a simple implementation of the interface. `Runner1` is annotated with `@Component` that does the bean wiring for the class. The next annotation is `@Order` which allows ordering of the runners in a multiple runners scenario. The highest order will be executed before others, hence the lowest will be the latest to execute. `org.springframework.core.Ordered` has two fields; HIGHEST_PRECEDENCE and LOWEST_PRECEDENCE (Integer.MAX_VALUE and Integer.MIN_VALUE respectively) to mark a class accordingly. Of course, it is possible to use integer values instead. If two runners have the same order or no order at all, the alphanumerical order of the class names will be used (Runner1 will be executed before Runner2 is executed).

## A CL Application with Sub-Commands ##
Let's think of a command line application which takes the first command line argument as the sub-command. For example, `git` 's first argument is a switch for executing a sub-command. `git branch` executes the branch related stuff while `git commit` is the commit logic of the tool. We want to implement something similar to this, and one of our goal is to make the sub-commands modular. We should be able to easily add new sub-commands without altering other sub-commands or the core of our application. The impact should be minimal.

{% gist amertkara/41b60c81fb4023d6a851 ApplicationStructure.txt %}

Of course, since each runner will be executed when its sub-command is passed, the order of the runners is not important. All runners can be considered at the same order.

![Multi Runners Project Structure](/public/images/2016/3/multi-runners-project-structure.png)

As you can see above, each sub-command (Parser, Analyzer) are under the subcommands package. New sub-commands can be added as they are implemented. This will ensure the modularity of the sub-commands, hence decoupling. Each sub-command has the entrance, which is a bean that implements the CommandLineRunner interface. 

{% gist amertkara/41b60c81fb4023d6a851 AnalyzerCommand.java %}

The implementation really depends on the application itself, but in this POC (Proof of Concept) application, I have two sub-commands that I can run `myapp parse [OPTIONS]` and `myapp analyze [OPTIONS]`. Above is the bean to handle the analyze commands. I have one option to parse the path for the analyzed file. The important bit is to check for the sub-command at the line 31 (self-explanatory).

That's all, the design for a command line application that has sub-commands to handle different tasks in a decoupled fashion.

## Code Repository ##
[Source](https://github.com/amertkara/multiple-runners)

## ERRATA ##
No corrections so far.