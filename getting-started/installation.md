---
description: Get up and running with ColdBox easily.
---

# Installation

**Welcome to the world of ColdBox!**

We are excited you are taking this development journey with us. Before we get started with ColdBox let's install CommandBox CLI, which will allow you to install/uninstall dependencies, start servers, have a REPL tool and much more.

## IDE Tools

ColdBox has the following supported IDE Tools:

* Sublime - [https://packagecontrol.io/packages/ColdBox Platform](https://packagecontrol.io/packages/ColdBox%20Platform)
* VSCode - [https://marketplace.visualstudio.com/items?itemName=ortus-solutions.vscode-coldbox](https://marketplace.visualstudio.com/items?itemName=ortus-solutions.vscode-coldbox)
* CFBuilder - [https://www.forgebox.io/view/ColdBox-Platform-Utilities](https://www.forgebox.io/view/ColdBox-Platform-Utilities)

## CommandBox CLI

![](../.gitbook/assets/commandboxlogo.png)

The first step in our journey is to [install](https://commandbox.ortusbooks.com/content/setup/installation.html) CommandBox. [CommandBox](https://www.ortussolutions.com/products/commandbox) is a ColdFusion (CFML) Command Line Interface (CLI), REPL, Package Manager and Embedded Server. We will be using CommandBox for almost every excercise in this book and it will also allow you to get up and running with ColdFusion and ColdBox in a much speedier manner.

> **Note** : However, you can use your own ColdFusion server setup as you see fit. We use CommandBox as everything is scriptable and fast!

### Download CommandBox

You can download CommandBox from the official site: [https://www.ortussolutions.com/products/commandbox#download](https://www.ortussolutions.com/products/commandbox#download) and install in your preferred Operating System (Windows, Mac, \*unix). CommandBox comes in two flavors:

1. No Java Runtime (80mb)
2. Embedded Runtime (120mb)

So make sure you choose your desired installation path and follow the instructions here: [https://commandbox.ortusbooks.com/content/setup/installation.html](https://commandbox.ortusbooks.com/content/setup/installation.html)

### Starting CommandBox

Once you download and expand CommandBox you will have the `box.exe` or `box` binary, which you can place in your Windows Path or \*Unix `/usr/bin` folder to have it available system wide. Then just open the binary and CommandBox will unpack itself your user's directory: `{User}/.CommandBox`. This happens only once and the next thing you know, you are in the CommandBox interactive shell!

![CommandBox Shell](<../.gitbook/assets/image (3).png>)

We will be able to execute a-la-carte commands from our command line or go into the interactive shell for multiple commands. We recommend the interactive shell as it is faster and can remain open in your project root.

{% hint style="info" %}
All examples in this book are based on the fact of having an interactive shell open.
{% endhint %}

## Installing ColdBox

To get started open the CommandBox binary or enter the shell by typing `box` in your terminal or console. Then let's create a new folder and install ColdBox into a directory.

```bash
mkdir myapp --cd
install coldbox
```

CommandBox will resolve `coldbox` from FORGEBOX ([www.forgebox.io](https://www.forgebox.io)), use the **latest version** available, download and install it in this folder alongside a `box.json` file which represents your application package.

```
Dir 0 Apr 25,2018 11:04:05 coldbox
File 112 Apr 25,2018 11:04:05 box.json
```

{% hint style="info" %}
You can also install the latest bleeding edge version by using the `coldbox@be` slug instead, or any previous version.
{% endhint %}

That's it! CommandBox can now track this version of ColdBox for you in this directory.

### Scaffolding ColdBox Applications

CommandBox comes with a `coldbox create app` command that can enable you to create application skeletons using one of our official skeletons or by creating [your own](../digging-deeper/recipes/application-templates.md) application template:

* **AdvancedScript** (`default`): A script based advanced template
* **elixir** : A ColdBox Elixir based template
* **ElixirBower** : A ColdBox Elixir + Bower based template
* **ElixirVueJS** : A ColdBox Elixir + Vue.js based template
* **rest**: A RESTFul services template
* **rest-hmvc**: A RESTFul service built with modules
* **Simple** : A traditional simple template
* **SuperSimple** : The bare-bones template

{% hint style="success" %}
You can find many scaffolding templates for ColdBox in our Github organization: [github.com/coldbox-templates](https://github.com/coldbox-templates)

Type `coldbox create app` help in CommandBox to get tons of help for scaffolding apps.
{% endhint %}

## Uninstalling ColdBox

To uninstall ColdBox from this application folder just type `uninstall coldbox`.

## Updating ColdBox

To update ColdBox from a previous version, just type `update coldbox`.
