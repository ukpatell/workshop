# Workshop Intruction

### This lab instructions are here to provide customization to [this](https://catalog.us-east-1.prod.workshops.aws/workshops/bdee9027-ee96-4651-a8d3-833c2a847206/en-US) workshop. As always, be mindful of this instructions. This are for educational purposes only, and not for production.


## Account Setup - AWS Console

## Modify Bedrock Access
0. Please make sure you’re in us-east-1
1. Click on “Model Access” on left bar, at bottom
2. Select Enable Specific Models at top
3. Select the following models:
    1. Titan Embeddings G1 - Text
    2. Titan Text G1 Express
    3. Titan Image Generator G1
    4. Tital Multimodal Embeddings G1
    5. Titan Text Embeddings V2
    6. Anthropic Claude 3 Sonnet
    7. Anthropic Claude 3.5 Sonnet v2
4. Scroll to bottom of page. Lower right corner at bottom - “Next”

![0-img-bedrock](./gifs/0-bmt-bedrock.gif)

## Create IAM User for CLI Access

In AWS Console, navigate to IAM

1. Select “Users” on left side of screen
2. Create User - pick a name (anything works)
    1. No console access required 
    2. Next
    3. Add User to Group
    4. Create Group - give a name
        1. Add AdministratorAccess to the new group by selecting it under Permission Policies
        2. Click Create User Group at bottom
    5. Select the new group to add the user to it
    6. Click Next
    7. Review - verify you are in the correct group - and click Create User
3. Select your new user on the Users page, by clicking on the name you created
    1. Create access key (upper-right)
    2. Use Case: CLI
        1. Confirm and Next
    3. COPY BOTH your Access Key and Secret Access Key somewhere safe
    4. Done

_EXAMPLE ACCESS KEY: AKIAUTFWSVB2D3HFFSMK_ <br>
_EXAMPLE SECRET: eTneQ8S6voUT9ovnrI/2XOoC/uFESH/vHVmGanVY_

![Image: 1-bmt-iam.gif](./gifs/1-bmt-iam.gif)

##  VS Code and AWS CLI Setup



1. From the main Workshop login page, select “Getting Started” → “Launching your ...“
2. Follow online instructions to start VSCodeConfigure EC2 Host in Terminal
3. In VS Code, install the ms-python Python extension

### Configure AWS CLI

1. VSCode: new terminal

```
aws configure
<follow prompts>
```

Please use the region **us-east-1** when you configure. Output format can stay **None**.

![Image: 2-bmt.gif](./gifs/2-bmt.gif)
## Project Code Download



#### Download the project files from GitHub

AWS Samples Link: https://github.com/aws-solutions-library-samples/guidance-for-a-multi-tenant-generative-ai-gateway-with-cost-and-usage-tracking-on-aws

In VS Code Terminal:


```
sudo yum install git
<enter password from workshop page>
<yes to all the things>

cd ~/workshop
git clone [https://github.com/aws-solutions-library-samples/guidance-for-a-multi-tenant-generative-ai-gateway-with-cost-and-usage-tracking-on-aws.gitmv guidance<...> bedrockcd bedrock](https://github.com/aws-solutions-library-samples/guidance-for-a-multi-tenant-generative-ai-gateway-with-cost-and-usage-tracking-on-aws.git)
```

#### Install the AWS CDK application

```
sudo npm -g install aws-cdk
<enter password from workshop page>
```

![3-BMT-Gitclone](./gifs/3-bmt-gitclone.gif)

## Open Project Folder in VS Code


Hamburger → File → Open Folder → Select the **bedrock** folder → OK



## Configure the Deployment

### Open setup/configs.json file

Explorer → setup/configs.json

Replace configs.json with the following. The differences are highlighted in red.


```

[
  {
    *"STACK_PREFIX"*: "<MY APP NAME PREFIX>",
    *"BEDROCK_ENDPOINT"*: "https://bedrock.us-east-1.amazonaws.com", 
    *"BEDROCK_REQUIREMENTS"*: "boto3>=1.35.38 awscli>=1.35.4 botocore>=1.35.38 **cryptography pytz**",
    *"LANGCHAIN_REQUIREMENTS"*: "aws-lambda-powertools langchain==0.3.3 langchain-community==0.3.2 pydantic PyYaml pytz",
    *"PANDAS_REQUIREMENTS"*: "pandas",
    *"SAGEMAKER_ENDPOINTS"*: "",
    *"VPC_CIDR"*: "10.10.0.0/16",
    *"API_THROTTLING_RATE"*: 10000,
    *"API_BURST_RATE"*: 5000
  }
]
```


Update <MY APP NAME PREFIX> with a unique string (your name, sid, something simple, etc.). 

Save and close **setup/configs.json** file.


### Satisfy  Requirements

1. Open requirements.txt
2. Add the following libraries to the end of the file, then save:

```
_cryptography_==44.0.2
_pytz_==2025.2
```


Now install the requirements:

Create a python virtual environment and install requirements into it:

```
python3 -m venv .venv 
source .venv/bin/activate
pip3 install -r requirements.txt
```

```

```

## Deploy



```
chmod +x ./deploy_stack.sh
./deply_stack.sh
```


Deploying the stack takes about 6-9 minutes. Coffee Break!!

![Img-BMT-deploy](./gifs/4-bmt-deploy.gif)

## Testing the Deployment


We need the public DNS address of our EC2 host, and to modify our security group (firewall) to allow an incoming connection to Jupyter Notebook. 


### Open a Network Port for Jupyter Notebooks

From the AWS Console, go to EC2. 


1. Select “Security Groups” on the left. 
2. Click the link for the Security Group ID whose description starts with “SG for VSCodeServer”.
3. Edit Inbound Rules
4. Add Rule
    1. Type: Custom TCP
    2. Port Range: 8888
    3. Source: My IP
5. Save Rules



### Copy the DNS address we need to reach the notebook

1. Select “Instances” on the left side of the console window
2. Check the box to the left of the VSCodeServer instance
3. Please copy the Public IPv4 DNS address on the right side of the Details Tab

![img-bmt-sg](./gifs/5-bmt-sg.gif)

### Configure Jupyter Notebook


By default, Jupyter Notebook only runs locally. We are running remote. We are going to install and start Notebook with remote access features turned on. Note that we previously configured a limites EC2 Security Group to permit network access from your IP to your EC2 host. 

Please consider for the sake of simplicity, that your remote connection to Notebook is not using TLS - do not place any sensitive information in a prompt. 

Please perform all work in a terminal.


#### Install Jupyter

```
pip3 install jupyter
```



#### Configure Jupyter

From the top-level of our project folder (“bedrock”)...


```
cd notebooks

jupyter notebook --generate-config
jupyter notebook password
**Set a password at the prompt.
**
vi /home/participant/.jupyter/jupyter_notebook_config.py


**Add the following lines to the config file, after the c = get_config()**** line:
**
c.NotebookApp.allow_origin = '*' #allow all origins
c.NotebookApp.ip = '0.0.0.0' # listen on all IPs
```

Save the config file by clicking: escape → :wq → enter


#### Start Jupyter

From a new terminal in VS Code (click the + in the upper-right of the terminal menu bar), start the server. 

There will be errors. Ignore them; they are caused by the VS Code proxy config in the terminal.

From the workshop directory (in a new terminal):


```
cd bedrock/notebooks
jupyter notebook
```


Do not select “Open in Your Browser” when the pop-up asks. It won’t break anything, but it will create a lot of network proxy errors in the console.


#### Connect to the Notebook

We will use the EC2 public DNS name we saved earlier. In your local web browser, connect to the notebook using the following URL - replacing the <example DNS name> with the one you saved:


```
[http://<ec2-54-87-149-89.compute-1.amazonaws.com>:8888/tree](http://ec2-54-87-149-89.compute-1.amazonaws.com:8888/tree)
```

Bypass secure connection error and Continue to site. Enter your password.

![img-bmt-jupyter](./gifs/6-bmt-jupyter-start.gif)


### Configuring your Notebook


Navigate and select the **01_bedrock_api.ipynb** notebook. We need to fill in these three elements under “Setting up API Url”:


```
api_url = "<API_URL>“
api_key = "<API_KEY>"
team_id = "<TEAM_ID>"
```



#### Getting your APIURL and API Key

In the AWS Console, navigate to API Gateway

1. On left hand bar, select “API keys”
2. Copy the text for the API key on the list. Use this for API_KEY



Getting your API URL

1. Click on APIs on the laft-hand bar
2. Select your API in the main window by clicking on the link
3. On the left-hand bar, select “Stages”
4. Copy the Invoke URL in the center panel on the main window
5. Use this as API_URL in the Jupyter Notebook Settings

#### Team ID

Use any string as your team id. Avoid whitespace or special characters (underscores and dashes should be fine). 


## Running the Notebook

You should be able to start running though the Notebook cells. Start at the first cell, initially. Run through the first two examples (list models and a chat example) to make sure things are working. After that you can skip around.

Some cells use models no longer in deployment in Bedrock, or that you might not have enabled. You may skip those.

The goal is to verify you have connectivity through the gateway, and produce some records for the usage tracking DB. You do not need to complete the notebook - though there are some example of note we will point out. The image generator and tool use examples highlight ways of interfacing with an LLM that might be new to you.

If you choose to modify code and test, you might want to modify the final two lines of most blocks that print a reponse, to better handle any errors that occur. Change the bolded text here::


```
response = requests.get(
    f"{api_url}/list_foundation_models",
    headers={
        "x-api-key": api_key,
        "team_id": team_id
    }
)

**text = response.json()[0]

print(text)**
```

With the following:

```
if response.ok:  # This checks if status code is 200-299
    text = response.json()[0]["generated_text"]
    print(text)
else:
    print(f"Error {response.status_code}: {response.reason}")
    error_detail = response.json() if response.content else "No error details provided"
    print(f"Error details: {error_detail}")
```


And if you want to pretty-print any json that comes back (such as the model list), try using a pattern like:

```
json_response = json.dumps(response.json()[0], indent=4, skipkeys=True)
print(json_response)
```


**Congratulations! You’ve deployed and tested the gateway.** 




