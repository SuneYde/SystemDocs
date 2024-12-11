# node-installation.md

Hey there! Let's set up your development environment together. Think of this like building your workshop - we want to make sure you have all the right tools in the right places. I'll explain not just what to install, but why each piece matters.

### Understanding Node.js and npm

Before we dive into installation, let's understand what Node.js is and why we need it. Imagine you're building a house - Node.js is like your foundation and basic framework. It lets you run JavaScript outside of a web browser, which is essential for:

- Running your React development server
- Managing project dependencies
- Running backend servers
- Executing build tools and scripts

### Step-by-Step Node.js Installation

Let's get Node.js installed properly on your system:

1. **Download Node.js**
   Head over to nodejs.org and you'll see two versions:

   - LTS (Long Term Support) - This is like the "stable" version
   - Current - This has the newest features but might be less stable

   For development work, I recommend the LTS version because it's more reliable and better supported by most packages.

2. **Installation Process**
   For Windows:

   ```bash
   # After downloading the installer:
   1. Run the .msi file
   2. Accept the license agreement
   3. Keep the default installation path (usually C:\Program Files\nodejs\)
   4. Let it install npm package manager
   5. Click through to finish
   ```

   For Mac (using Homebrew):

   ```bash
   # If you don't have Homebrew, install it first:
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

   # Then install Node:
   brew install node
   ```

3. **Verify Installation**
   Open your terminal/command prompt and run:

   ```bash
   node --version
   npm --version
   ```

   You should see something like:

   ```bash
   v18.17.0  # Node version
   9.6.7     # npm version
   ```

### Setting Up Your Development Tools

Now that we have Node.js, let's set up some essential development tools:

1. **Install Git**
   Windows:

   - Download from git-scm.com
   - During installation, choose these options:
     ```bash
     Use Git from Git Bash only
     Use the OpenSSL library
     Checkout Windows-style, commit Unix-style line endings
     Use MinTTY
     Enable file system caching
     ```

   Mac:

   ```bash
   brew install git
   ```

2. **Configure Git**

   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

3. **Install a Code Editor (VS Code)**
   - Download from code.visualstudio.com
   - Install these essential extensions:
     - ES7+ React/Redux/React-Native snippets
     - Prettier - Code formatter
     - ESLint
     - GitLens
     - Auto Rename Tag

### Setting Up Your First Project

Let's create a test project to make sure everything works:

```bash
# Create a new directory
mkdir my-first-project
cd my-first-project

# Initialize a new Node.js project
npm init -y

# Install React and necessary dependencies
npm install react react-dom next

# Create a test file
echo "console.log('Hello from Node.js!')" > test.js

# Run it
node test.js
```

If you see "Hello from Node.js!" in your terminal, congratulations! Your environment is working.

### Environment Variables

Environment variables are like secret settings for your project. Let's set them up properly:

1. Create a `.env` file in your project root:

   ```bash
   touch .env
   ```

2. Add this to your `.gitignore`:

   ```bash
   # Create .gitignore if it doesn't exist
   echo "node_modules/" > .gitignore
   echo ".env" >> .gitignore
   ```

3. Create a template for others:
   ```bash
   # Create .env.example
   echo "DATABASE_URL=mongodb://localhost:27017
   API_KEY=your_api_key_here
   PORT=3000" > .env.example
   ```

### Project Structure Setup

Let's create a good folder structure for your MERN projects:

```bash
mkdir -p src/{components,pages,styles,utils,api}
touch src/styles/globals.css
touch src/pages/index.js
```

This creates:

```
project/
├── src/
│   ├── components/
│   ├── pages/
│   ├── styles/
│   ├── utils/
│   └── api/
├── .env
├── .env.example
├── .gitignore
└── package.json
```

### Setting Up MongoDB Locally

1. **Install MongoDB**
   Windows:

   - Download MongoDB Community Server from mongodb.com
   - Run the installer
   - Choose "Complete" installation
   - Install MongoDB Compass (the GUI)

   Mac:

   ```bash
   brew tap mongodb/brew
   brew install mongodb-community
   ```

2. **Start MongoDB**
   Windows:

   - MongoDB runs as a service automatically

   Mac:

   ```bash
   brew services start mongodb-community
   ```

### Testing Your Complete Setup

Let's make sure everything works together:

1. Create a basic Express server:

   ```javascript
   // server.js
   const express = require("express");
   const app = express();

   app.get("/", (req, res) => {
     res.json({ message: "Environment is ready!" });
   });

   app.listen(3000, () => {
     console.log("Server running on http://localhost:3000");
   });
   ```

2. Run it:

   ```bash
   node server.js
   ```

3. Visit http://localhost:3000 in your browser. If you see the message, everything is working!

### Troubleshooting Common Issues

If you run into problems, here are some common solutions:

1. **Node Command Not Found**

   ```bash
   # Windows: Add to PATH manually
   # Mac/Linux:
   export PATH=$PATH:/usr/local/bin/node
   ```

2. **Permission Errors with npm**

   ```bash
   # Mac/Linux:
   sudo chown -R $USER /usr/local/lib/node_modules
   ```

3. **MongoDB Won't Start**
   ```bash
   # Check if the port is in use
   lsof -i :27017
   # If something's using it:
   sudo kill <PID>
   ```

Need help with any of these steps? Want to set up additional tools? Just let me know! Remember, a well-set-up environment makes development much more enjoyable and efficient.
