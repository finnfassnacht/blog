# Deploying a Simple HTTP Server to IBM Code Engine from Source using Python, Node.js and Go
In this blog, we will explore how to deploy a simple HTTP server to IBM Code Engine using three popular programming languages: Python, Node.js, and Go. We will start by explaining what IBM Code Engine is and how to set it up. We will then demonstrate how to use the web UI and the CLI to deploy our code. By the end of this blog, you will have a clear understanding of how to deploy your own code to IBM Code Engine using your preferred programming language.
## Setting up Code Engine
Before we can deploy our simple HTTP server, we need to set up IBM Code Engine. If you don't have an IBM Cloud account yet, you can create one for free [here](https://cloud.ibm.com/codeengine/overview).

After that you have two options: using the web UI or the CLI. 

#### Using the CLI

To set up IBM Code Engine using the CLI, follow these steps:

1.  Install the IBM Cloud CLI
>on Linux
```
curl -fsSL https://clis.cloud.ibm.com/install/linux | sh
```
>on MacOS
```
curl -fsSL https://clis.cloud.ibm.com/install/osx | sh
```
>on Windows
```
iex (New-Object Net.WebClient).DownloadString('https://clis.cloud.ibm.com/install/powershell')
```

2.  Log in to IBM Cloud
```
ibmcloud login
```
3.  Install the Code Engine plugin
```
ibmcloud plugin install code-engine
```

#### Using the Web UI
To set up IBM Code Engine using the Web UI, follow these steps:
1. Go to  [Code Engine](https://cloud.ibm.com/codeengine/overview)
2. Log in

Once you have set up IBM Code Engine, you can proceed to deploy your simple HTTP server. In the next section, let's look at some sample code for your HTTP server in Python, Node.js and Go.

## Sample code

The following sample code will start a server on port 8080 and set up a simple GET route that serves a "Hello World" message. The beauty of using IBM Code Engine is that you don't need any specific modules or configurations. If the code runs successfully on your localhost at port 8080, it will most likely run on Code Engine without any modifications.

**Nodejs app with Express.js**

```js
// require expressjs
const express = require("express")
const app = express()
PORT = 8080
app.use(express.json())
// use router to bundle all routes to /api
const router = express.Router()
app.use("/api", router)

// get on root route (/api)
router.get("/", (req,res) => {
	res.json({"msg":"hello world!!!"})
})

// start server
app.listen(PORT, () => {
	console.log("Server is up and running!!")
})
```

Find the Github Repository [here](https://github.com/finnfassnacht/Code-Engine-nodejs).

**Golang**
```go
package main
import (
	"fmt"
	"log"
	"net/http"
)

func helloworld(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}

func main() {
	http.HandleFunc("/", helloworld)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```
Find the Github Repository [here](https://github.com/finnfassnacht/Code-Engine-go).


**Python**
```python
from flask import Flask
import os

app = Flask(__name__)

@app.route("/")
def hello_world():
	return "Hello World"

# Get the PORT from environment
port = os.getenv('PORT', '8080')
if __name__ == "__main__":
	app.run(host='0.0.0.0',port=int(port))

```
it is important to note that Python will require a Procfile: 
```
web: python main.py
```
Find the Github Repository [here](https://github.com/finnfassnacht/Code-Engine-python).

## Deploy an Application

Now that we've looked at the sample code, let's talk about deploying it. The first step is to create a new project. Your application will be contained within this project.

**To create a new Project in the Web UI** 

1. Click on projects 
2. Click on Create
3. Select a desired Location .i.e us-east
4. Pick a name for your Projects
5. Resource group set to Default 

**To create a new Project using the CLI** 
```
ibmcloud ce project create --name projectname
```
Note that group Default and region us-east is the default,
if you want to target a specific region use 
```
ibmcloud target -r eu-gb
```
or if you want to taget a specific group use
```
ibmcloud target -d Default
```


Now that we've created a project, let's move on to deploying our code. We'll be deploying our code from a source that's hosted in a [Github Repository](https://github.com/finnfassnacht/Code-Engine-nodejs), along with the package.json and package-lock.json files. In this repository, you'll find a sample server written in Node.js (see the sample code above).

**In the Web UI**
1. Click on your project in the Code Engine Dashboard.
2. Select "Applications" from the sidebar.
3. Click "Create" to create a new application.
4. Select "Source code" and enter the repository link.
5. Click the "Specify build details" button.
6. If your repository is not private, you can proceed to the next step.
7. Select "Cloud Native Buildpack" and click "Next".
8. Unless reason to change something click "Next" again
9. Once you're done configuring the build options, click "Create" to deploy your application.

Blast off!

**In the CLI**
```
ibmcloud ce app create --name appname --src repolink-here --str buildpacks
```
>Define the name of your App
```
--name appname
```
>Define the source of the code 
```
--src repolink or /path/to/file
```
note that the source can be a local file as well 
>Define stratagey
```
--str buildpacks
```

## Update an Application
Lets also briefly talk about how to update a application 
I added a simple POST route to my nodejs script.
```js
router.post("/return", (req,res) => {
	let D = (req.body)
	res.send("Returned >>" + D.data)
	res.end()
})
```
## How it works
That's all you need to do to deploy your app from source. Let's talk about the magic that happens after you hit "Create."

First, IBM Code Engine loads your source code from a repository or from your computer locally. It then builds a Docker image and runs it on the IBM Cloud.

Your container is run using Kubernetes, which automatically scales your application up and down based on traffic. This ensures that your app always has enough resources to handle incoming requests, while saving money during low traffic periods when Kubernetes scales down your application.

## Better Performance
Ok so if you want to deploy a simple web app fast and easy, no you know how.
However what if i told you there was a way to make your app a lot faster?
Enter Containers, if you deploy your app from source, Code Engine will make a container (and image) for you.
The language you use will be detected and a broad pre built image will be selected for it,
altough Code Engine knows what you need, it doesn't know what specifically you dont need.
This results in larger images and there fore wourse performance because it takes longer to load.
You however know what your code needs and more importantly doesn't need. We can choose a lighter base image and there for get better performance.
