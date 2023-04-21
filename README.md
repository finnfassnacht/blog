# Deploying a Simple HTTP Server to IBM Cloud Code Engine from Source Code using Python, Node.js and Go

In this blog post, we will explore how to deploy a simple HTTP server to Code Engine using three popular programming languages: Python, Node.js, and Go. IBM Cloud Code Engine is a fully managed, serverless platform that runs your containerized workloads, including web apps, microservices, event-driven functions or batch jobs. 

In this post we will focus on Code Engine applications, which are designed to serve HTTP requests.  We will demonstrate how to use the web UI and the CLI to deploy our code. By the end of this post, you will have a clear understanding of how to deploy your own code to Code Engine using your preferred programming language.

## Setting up Code Engine

Before we can deploy our simple HTTP server, we need to get set up Code Engine. If you don't have an IBM Cloud account yet, you can create one for free [here](https://cloud.ibm.com/codeengine/overview). Be aware that Code Engine requires an account with a valid credit card on file. However, Code Engine provides a generous free tier with ample resources to kickstart your project.

After logging into the IBM Cloud, you have two options to interact with the IBM Cloud and Code Engine: using the command line interface (CLI) or the web UI. 

#### Using the CLI

Here are the steps:

1.  Install the IBM Cloud CLI
>On Linux
```
curl -fsSL https://clis.cloud.ibm.com/install/linux | sh
```
>On MacOS
```
curl -fsSL https://clis.cloud.ibm.com/install/osx | sh
```
>On Windows
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
To use the Web UI, follow these steps:
1. Go to  [Code Engine](https://cloud.ibm.com/codeengine/overview)
2. Log in

You can now proceed to deploy your simple HTTP server. In the next section, let's look at the sample code for your HTTP server in Python, Node.js and Go.

## Sample code

The following sample code will start a server on port 8080 and set up a simple GET route that serves a "Hello World" message. The beauty of using Code Engine is that you don't need any specific modules or configurations. If the code runs successfully on your localhost at port 8080, it will run on Code Engine without any modifications.

**Nodejs app with Express.js**

```js
// require expressjs
const express = require("express")
const app = express()
// define port 8080
PORT = 8080
app.use(express.json())
// use router to bundle all routes to /
const router = express.Router()
app.use("/", router)
// get on root route
router.get("/", (req,res) => {
	res.send("hello world!!!")
})

// start server
app.listen(PORT, () => {
	console.log("Server is up and running!!")
})
```

Find the Github repository [here](https://github.com/IBM/CodeEngine/tree/main/helloworld-samples/app-nodejs).

**Golang**
```go
package main
import (
	"fmt"
	"log"
	"net/http"
)
// create function for route
func helloworld(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}

func main() 
	// use helloworld on root route
	http.HandleFunc("/", helloworld)
	// use port 8080
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```
Find the Github repository [here](https://github.com/IBM/CodeEngine/tree/main/helloworld-samples/app-golang).


**Python**
```python
from flask import Flask
import os

app = Flask(__name__)

# set up root route
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
Find the Github repository [here](https://github.com/IBM/CodeEngine/tree/main/helloworld-samples/app-python).

## Deploy an Application

Now that we've looked at the sample code, let's move on to deploying it. The first step is to create a new project. Your application will be contained within this project.

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
ibmcloud target -r us-east
```
or if you want to taget a specific group use
```
ibmcloud target -d Default
```

Now that we've created a project, let's move on to deploying our code. We'll be deploying our Node.js code from a source that's hosted in a [Github repository](https://github.com/IBM/CodeEngine/tree/main/helloworld-samples/app-nodejs), along with the package.json and package-lock.json files. In this repository, you'll find a sample server written in Node.js (see the sample code above).

**In the Web UI**
1. Click on Projects in the Code Engine dashboard.
2. Select your project.
3. Select "Applications" from the sidebar.
4. Click "Create" to create a new application.
5. Select "Source code" and enter the repository link.
6. Click the "Specify build details" button.
7. If your repository is not private, you can proceed to the next step.
8. Select "Cloud Native Buildpack" and click "Next".
9. Unless you have reason to change something click "Next" again.
10. Once you're done configuring the build options, click "Create" to deploy your application.

Done! You can now click on the provided link to see your application live.

**In the CLI**
```
ibmcloud ce app create --name appname --src repolink-here --str buildpacks 
```
Define the name of your App
```
--name appname
```
Define the source of the code 
```
--src repolink or /path/to/folder
```
note that the source can be a local file as well 
Define stratagey
`
--str buildpacks
`
When your code is not located in the root directory of your repo or directory, it is important to specify the exact location of your code.

Use
`
--build-context-dir
`
to define your
`
/path/to/folder
`
To run the example code in the CodeEngine repository (as I am doing), execute the following command:
```
ibmcloud ce app create --name appname --src  https://github.com/IBM/CodeEngine.git --str buildpacks --build-context-dir /helloworld-samples/app-nodejs/
```

## Update an Application
Let's briefly discuss how to update your application. For instance, you might want to add a new POST route to your Node.js code that returns data sent to the server.
```js
router.post("/return", (req,res) => {
	let D = (req.body)
	// return data back to user
	res.send("Returned >>" + D.data)
	res.end()
})
```
**Using the CLI**
```
ibmcloud ce app update --name appname --src repolink-here --str buildpacks
```
**Using the Web UI**
1. Navigate to your Application
2. Select "Configuration"
3. Click "Edit and create new revision"
4. After the Image is built hit "Save and Create"

## How it works
That's all you need to do to deploy your app from source. Let's talk about the magic that happens after you hit "Create."

First, Code Engine loads your source code from a repository or from your computer locally. It then builds a container image and runs it on IBM Cloud.

Specifically, your container is managed by two open source technologies (Kubernetes and Knative), which automatically scale your application up and down based on the traffic it receives. This ensures that your application always has enough resources to handle incoming requests, while saving money during low traffic periods when your application is scalled down again. If it receives no traffic, it will even scale to 0 and stops incurring charges. Once your application receives traffic, it will automatically "wake up" and scale back up to one instance.

## Better Performance
Congratulations, you now know how to deploy a simple web app quickly and easily! 

But what if I told you there's a way to make your application more efficient? Enter containers. 

When you deploy your app from source, Code Engine creates a container image for you. The platform detects the language you're using and selects a general purpose pre-built image for it. But of course, it doesn't know exactly what your app needs or doesn't need. As a result, the image may be larger than necessary, leading to slower deployment performance and longer start up times. By defining your own, container image, you can optimize your app by ensuring that only the necessary components are included.

## Making an Image

Since Code Engine will build the container image for you, you don't need to worry about installing Docker or getting its daemon to run properly.

To build your own custom image, you'll need to create a Dockerfile. In this example, we'll be creating one for the Node.js application. To get started, create a new file called "Dockerfile" without any file extension. 

```dockerfile 
FROM node:alpine
WORKDIR /usr/src/app
COPY package*.json index.js ./
RUN npm install
EXPOSE 8080
CMD ["node", "index.js"]
```
Use a pre-built image (alpine is extremely lightweight)
```dockerfile
FROM node:alpine

```
Specify a working directory
```dockerfile
WORKDIR /usr/src/app
```
Copy package.json and package-lock.json
```dockerfile
COPY package*.json index.js ./
```
Install the required packages
```dockerfile
RUN npm install
```
Expose the port 8080 to the outside
```dockerfile
Expose 8080
```
Finally run the server
```dockerfile
CMD ["node", "index.js"]
```

Once you have configured your Dockerfile (and added it to your repo or folder) you can easily deploy the image.

**Using the WebUI**

Deploying your code with a Dockerfile in the Web UI works the same as deploying your code without one. However, in step 8, you will need to select "Dockerfile" so that Code Engine uses the instructions in your Dockerfile to build the image.

**Using the CLI**

```
ibmcloud ce app create --name appname --src repo or /path/to/folder
```

Note that it's important that your Dockerfile is located in the root directory of your project.

By specifying how to build the image, the resulting image size can be dramatically reduced. In my case, I was able to reduce the image size by about 90%. This means that everything will run much faster.

## In conclusion

Deploying a simple HTTP server to Code Engine is a straightforward process that can be done using different programming languages such as Python, Node.js, Go and more. With Code Engine, you can deploy your code without worrying about the underlying infrastructure, making it easy to focus on writing your code. 

In this blog post, we explored how to set up Code Engine using the web UI and the CLI and then provided sample code for the three programming languages. We also demonstrated how to create a new project and deploy your code. With the steps outlined in this post, you should have a clear understanding of how to deploy your own code to Code Engine using your preferred programming language and method. 

Try it out for yourself and see how easy it is to deploy your code to IBM Cloud Code Engine.
