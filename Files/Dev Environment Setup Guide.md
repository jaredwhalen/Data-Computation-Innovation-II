# **Dev Environment Setup Guide**

**1\. VS Code**

Download and install Visual Studio Code: [https://code.visualstudio.com](https://code.visualstudio.com)

Once installed, add these two extensions (search in the Extensions panel — View \> Extensions):

* **Prettier**  
* **Svelte for VS Code**

---

**2\. GitHub**

Create a free account at [https://github.com](https://github.com) if you don't already have one.

Then install Git and connect it to your GitHub account using the instructions here: [https://docs.github.com/en/get-started/git-basics/set-up-git](https://docs.github.com/en/get-started/git-basics/set-up-git)

---

**3\. Node**

We'll install Node using nvm (Node Version Manager), which lets you manage multiple versions of Node without conflicts.

Run the install command from [https://nodejs.org/en/download](https://nodejs.org/en/download)

Confirm it worked:

```
which node
```

You should see a path containing `.nvm`. If you see `command not found`, close and reopen your terminal and try again.

**4\. The class template**

Clone the starter project to your machine by going to the [repo](https://github.com/jaredwhalen/class-svelte-starter) and clicking the green “Use this template” button.

Open the folder in VS Code:

```
cd class-svelte-starter
code .
```

Install dependencies:

```
npm install
```

Start the dev server:

```
npm run dev
```

Open your browser and go to the URL shown in the terminal. If you see the starter page, you're all set.