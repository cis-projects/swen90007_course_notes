# Milestone -1: Tools

We're planning to uses a fair few technologies to build this system, React, PostgreSQL, Java Servlet API, and others; so let's start with installing the main ones.

## Node.js runtime 

We're developing the UI with React, so we'll first need a Node.js runtime. If you already have Node.js on your local system make sure that you're using the version required below. Otherwise, follow the steps below to install the Node.js runtime. 

### Install

The best way to install Node.js is with the [Node Version Manager (NVM)](https://github.com/nvm-sh/nvm). NVM allows you to easily switch between versions of Node.js; which, may not be hugely beneficial for this project, but is often required when working on multiple existing code bases - some of which may target different versions of Node.js. If you are developing on macOS or Linux, installing NVM is easy and recommended; if you are working on Windows, NVM may not  be well supported and you may need to instead install Node.js directly. Regardless of your OS follow the [NVM install guide](https://github.com/nvm-sh/nvm) for your system, falling back to a direct Node.js install - as detailed on the [official Node.js site](https://nodejs.org/en/download/package-manager/) - if advised.

At the time of authoring this document Node.js LTS (Long Term Support) was version [18.14.0](https://nodejs.org/en/download/). installing this version and its associated NPM version is simple with NVM. If your system is not well supported by NVM, and are required to instal Node.js directly, make sure you also target this version.

Install Node.js with NVM:

```shell
nvm install 18.14.0
nvm use 18.14.0
```

### Verify 

Once you've installed Node.js you can verify your installation like so:

```shell
node --version
```

Node.js ships with both the Node Package Manager (NPM) and Node Program Executor (NPX). Verify those have been installed too (note that the versions for both NPM and NPX wont be the same as your Node.js installation, for example: Node.js v18.14.0 ships with NPM v9.3.1).

Verify that NPM is installed:

```shell
npm --version
```

Finally, verify that NPX is installed:

```shell
npx --version
```

> [!question] What are NPM and NPX?
> NPM is primarily a package manager for Node.js dependencies (called modules), if we want to make use of external libraries - such as React - we can use NPM to fetch and manage those for us. NPM can also be used to run scripts, and is handy for automating tasks within your project - more on that later. NPX allows us to execute arbitrary node modules, normally you'll do this if you want to run a module managed by NPM, but NPX is not restricted to modules you've installed explicitly, for example `npx https://gist.github.com/Tynael/0861d31ea17796c9a5b4a0162eb3c1e8` will run a module located on GitHub. 

## Java and Tomcat

The Set up for Java is identical to the set up detailed [here] with a few minor differences

- [ ] #daily #work/unimelb differences for Java setup ⏳ 2023-08-05

## PostgreSQL and pgAdmin

- [ ] #daily #work/unimelb differences for PostgreSQL setup ⏳ 2023-08-05

## Visual Studio Code

Visual Studio Code (or simply - VS Code) is the IDE used in this guide for developing the UI codebase. Download the latest version of VS Code [here](https://code.visualstudio.com/).
