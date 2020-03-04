---
layout: post
title: "Reactception : extending a VS Code extension with Webviews and React"

author: Ciaanh
categories: development
tags: VScode extension react webview
toc: true
---

VS code has become quite popular among coders these days. Being very open to modifications using "extension" really helped to expand the possibilities of what you can do with VS Code.
Lately I was experimenting with webviews and I had one of this "what if" moment :
what if we put a React application inside a webview inside an extension inside VS Code

Here begins this little inception experiment, a small walkthrough of what we can (but probably shouldn't, more on that later) do with VS Code extensions, webviews and React.

## Setup the project

Every journey has a start point, ours will be a need to edit a very complex config file in Json. We would like to provide a small GUI to edit the file, make some checks and so on.
Here is our small config file :

{% gist 8afcf0b8cc0f1e51d6572e4ba0a9ad91 %}

We can now begin to code and create our project, the initial setup is quite easy:

- first install vs code and node.js if you don't already have them
- in order to create a default extension you can use Yeoman templates : npm install -g yo generator-code
- create the extension project with : yo code

You will be asked several questions to prepare everything, I chose a Typescript all along this project project but this up to you.

> ? What type of extension do you want to create? New Extension (TypeScript)  
> ? What's the name of your extension? vscode-react  
> ? What's the identifier of your extension? vscode-react  
> ? What's the description of your extension? Inception  
> ? Initialize a git repository? Yes  
> ? Which package manager to use? npm  

Open the project in VS Code and here we have a basic "Hello world !" extension that you can test by pressing F5 to enter debug mode which will launch a new VS Code window where the extension is available, just press Ctrl+maj+P to display the command palette and type in Hello World.
  
 ![Hello world](/posts_content/hello_world.gif)

(you can find this part in the official startup guide for [VS Code extensions](https://code.visualstudio.com/api/get-started/your-first-extension) )

## Adding a language support to the extension

The goal of our extension is to edit config.json files, therefore we want to be able to use the extension only with Json files.

We will first edit the pacjage.json file which is the global declaration of our extension to VS Code.
In the "contributes" section we add the language part to declare our dummyconfig language.

{% gist 55188d8302357881d2294932a01306bf %}

As we are no longer making a Hello world extension we can rename the command that we will use replace it with the following declaration

{% gist 29f3b26e91587ba0bf72e2e3d61f8dd5 %}

We also need to rename the corresponding activationEvents

{% gist a919d93f6a9f2b36a3b53aa508e360ca %}

With this configuration we can now begin to code our extension. First we change the action to which our code will be attached in the src/extension.ts file.

Now in our extension, instead of showing "Hello world", we will prompt the user to select a dummyConfig file (just a Json as we previously declared) and display the path of this file as an information message. VS Code API expose a function ["window.showOpenDialog"](https://code.visualstudio.com/api/references/vscode-api#window.showOpenDialog) which take an "OpenDialogOptions" object to define the dialog properties.

{% gist 5cd4e371af539322682b290ff40278e7 %}

You can test the new command as we did before by launching debug but this time using the modified command name "Webview React"

## Displaying a basic webview

As we are now able to select the file we want to edit we can start adding a GUI using the Webviews.
The [api documentation for webviews](https://code.visualstudio.com/api/extension-guides/webview) is very well done and could help you if you need more details for this part .

For now we will begin by creating a folder named view and a typescript file ViewLoader.ts where we will manage our webview.
In this class we use "createWebviewPanel" to create the webview and then set the content of the webview with "\_panel.webview.html".
  
{% gist 476b35ecc111b9966f50841fc641c054 %}

Before testing the basic webview we just need to call the ViewLoader in our extension where we handle the command instead of calling "showInformationMessage".
  
{% gist 2bdc3617ea2dadb47f26dbf17a862d3e %}

And now we can debug our extension and see the webview content.
  
 [webview-demo.gif]

## Create a react application inside our extension project

Here is the tricky part : How to manage an extension for VS Code and a React application independently inside the same project ?
All the magic will come from npm and webpack with the help of some plugins ;-)

First thing to do is to include in our project all the libraries that we will need :

- React, obviously - ```npm install react react-dom```
- As I use Typescript I need to include the types but you might not if you use classic JS - ```npm install --save-dev @types/react @types/react-dom```
- In order to build our React application we will use webpack - ```npm install --save-dev webpack webpack-cli```
- And some tools, ts-loader obviously for Typescript, style and css loader for... style and css. And [npm-run-all](https://www.npmjs.com/package/npm-run-all) will allow us to chain the build task for the extension and for the react app - ```npm install --save-dev ts-loader style-loader css-loader npm-run-all```

We can now create the folder for our react application, let's name it "app" and place it inside the view folder. Then create the files "index.css" "index.tsx" and "tsconfig.json" in the "app" folder.
In the tsconfig file we declare that we use react and to use a "configViewer" folder as an output directory at the root of our extension project, this folder will be useful later to bind the extension and the react application.
  
{% gist 50f639e629a27c7fcff6462e90585722 %}

We need to update the package.json of our extension to declare the two compilations tasks we now have, one for the extension and one for the react application.
We change the "compile" and "watch" tasks and add several sub-tasks
  
{% gist 6ec6c35059290a80a9051ec9a6689947 %}

At the root of our extension we add the configuration file for webpack, "webpack.config.js". No tricky part here, we declare the usage of our different loader, we define the entry point of the react application and we set up again the output folder of our react application as the "configViewer" folder.
  
{% gist 64e427402c9afc06f91067f5a5836644 %}

Edit the root "tsconfig.json" to exclude the react application from the scope : "**/view/app/**"

{% gist 81202492d34a3d934b4b3c840e53c094 %}

Edit the .gitignore to add an entry for "configViewer/configViewer.js" in order to ignore the compiled file of our react application and finally add the content in the "index.tsx" to have a basic react application.
  
{% gist 701f2b563b5d6998381f9bf1bd97b421 %}

## Use the react application inside the webview

As I am using Typescript you can guess that I like to type everything. That is true also for the configuration file we are trying to edit.
We will introduce a model.ts file which will contain the interfaces for our configuration types and place it in the view/app folder.
  
{% gist 7567fe4808ceaa5ffa75762f6e408ae3 %}

As we will be using this file inside the React application but also inside our extension we have to tell our typescript compiler to include this file. In the tsconfig.json for the extension (in the root folder) we have to edit the exclusion part to exclude content of the view/app folder but not the model.ts file, this will allow us to use it in the extension.
  
{% gist f1334869222dc4c4e1939f4e097750b2 %}

In our Viewloader we can now load our file and deserialize its content in our class constructor:
  
{% gist fc041cbafe487e1acacaa5e34954d89d %}

Now that we have the content to display we can tell our webview to load the React application. As webpack compile our application as a js file all we have to do is to allow execution of javascript in our webview and reference the file to be loaded.
First we need to tell the webview where the js file to load is located, to do so we need to have the current path of the extension and pass it to the getWebviewContent function (we get it from the context of our extension).

Enable script is a parameter when we create the webview where we also give the authorized path to load the scripts from.
  
{% gist b85a038f1f5a13ce5fbcd01252d2bb17 %}

We also need to declare the file types authorization inside the webview template by defining in the header of the webview the Content-Security-Policy

Now that we authorized javascript execution we can load the configViewer.js file in the webview and pass the configuration file content too as a javascript variable window.initialData
  
{% gist 273500932d42dcbb24431f8bcdc37b37 %}

Notice that we also defined a variable window.acquireVsCodeApi which is a function exposed by VS Code to allow communication between VS Code and the webview. Indeed we have to manage the lifecycle of the webview as everytime the tab lose the focus in VS Code the webview will be destroyed and recreated when the focus comes back.

The "vscodeApi" in our small application will allow us to manage a state of the webview which allow us to persist data when the webview is destroyed and recreated.
To manage this lifecycle we must pass these variables to our application.

In the index.tsx we add an interface to declare these variables and execute the acquireVsCodeApi function.
These variables will be passed in the component that we will now create to display the content of the configuration, let's call it "config.tsx".

The first thing we want this component to do is to manage the state and display some basic content so in the constructor of our component we look at the initial data from the props or the VS Code api state for data to use and set it in the component state.
And finally we can put something in our render function to display the list of users and other data from our configuration file and use the component in our index.tsx file.
  
{% gist f7bab5711c5f6b1e2e6882f8e19278f9 %}
  
{% gist 9ae051a501eba76b25f938d1a4bd0bf6 %}

To make things a little bit prettier let's add somme CSS in our index.css file

{% gist 26c5b7a95de6a57a8fd6cd75ffc1d65c %}

## How to make the React application communicate

VS Code api exposes a way to send "commands" to and from the webview. It is also possible to call specific VS Code action using URI but we will not talk about this part here.
The action we would like VS Code to do for us is obviously to save the configuration once we are done editing it but for now we can only display the configuration and not edit it, let's solve this.
We will focus only on user management and add to behavior in our React application : manage roles and users.

In our configuration file we have a list of users so we could add a user. For each user we have a "isActive" boolean already mapped to a checkbox so we can edit it and we have a list of roles, here again we will be able to add a role to a user.

We add a function onChangeUserActiveState to handle... change on the isActive checkbox and two text inputs and their corresponding function to add auser or a role.

As we previously talk about, to manage webviews lifecycle we need to update a second "state" so let's create a function for this:

{% gist a46eafd0e5a0a76e6932aaae71fa230d %}

Now that we can edit our configuration saving it is quite simple, we need to define a type for the messages we will exchange, send the message when we hit a "save" button and then intercept the message in our ViewLoader to execute the command.

In our common model we add an interface ICommand which will tell us what type of command we want to do (saving the file) and also contain the corresponding data. To know the type of command to execute we also declare a CommandAction enum which is pretty straightforward.

In the React application we add a "save" button and the corresponding event handler saveConfig which create the command adn just call VS Code Api with function postMessage, quite easy.

{% gist 77ff88e35244a2e7d34516c9da0ae002 %}

And the last part is of course to handle the command. In the ViewLoader we "subscribe" to message from the webview by calling the function onDidReceiveMessage on the webview class.
When calling the "save" command we will write the new content of the file and display an information message to confirm the action.

{% gist 834756ba4b99ae02a8daf28c998894d4 %}

{% gist 71c3335cf1f25b643c01f5335426c303 %}

## Conclusion

The webviews implementation in VS Code make it really easy to add a custom interface for your extension and with the magic tricks with webpack even using React is really simple.
But as I warned at the begining of this short journey, and as Professor Ian Malcolm (Jeff Goldblum) said "you were so preoccupied wether or not you could that you didn't stop to think if you should".
And that is the main problem, webviews can take a lot of resources so we should be really careful when using it, especialy if we inject a React application inside it. Always ask if what you are doing can't already be done in VS Code using the API.
Anyway here you have just an example of what can be done with VS Code and webviews but many others ideas are possible like displaying a unified view to edit i18n localisation in an Angular application which could extend the [extension created by Oleksandr Reznichenko](https://medium.com/younited-tech-blog/my-way-to-vs-code-extension-cdf84cdb36ba).

Source code of every step can be found on [Github](https://github.com/Ciaanh/reactception)
