﻿This sample shows how to integrate QnA Maker in a simple bot with ASP.Net Core 2 and Application Insights. 

# Concepts introduced in this sample

[Dispatch](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) is a tool to create and evaluate LUIS models used for NLP (Natural Language Processing). 
Dispatch works across multiple bot modules such as LUIS applications, QnA knowledge bases and other NLP sources (added to dispatch as a file type). 
Use the Dispatch model in cases when:
* Your bot consists of multiple modules and you need assistance in routing user's utterances to these modules and evaluate the bot integration.
* Evaluate quality of intents classification of a single LUIS model.
* Create a text classification model from text files.

The [Language Understanding Intelligent Service (LUIS)](https://luis.ai), is a machine learning-based service to build natural language into apps, bots, and IoT devices. 
LUIS allows to Quickly create enterprise-ready, custom models that continuously improve. 

The [QnA maker Service](https://www.qnamaker.ai) enables you to build, train and publish a simple question and answer bot based on FAQ URLs, structured documents or editorial content in minutes.

The [Application Insights](https://azure.microsoft.com/en-us/services/application-insights/) enables you to discover actionable insights through application performance management and instant analytics.

In this sample, we demonstrate how to use the Dispatch service to route utterances when there are multiple LUIS models and QnA maker services for different scenarios supported by a bot. 
In this case, we confiure dispatch with multiple LUIS models for conversations around home automation and weather information, plus QnA maker service to answer questions based on a FAQ text file as input.

# To try this sample

- Clone the samples repository
```bash
git clone https://github.com/Microsoft/botbuilder-samples.git
```
## Install BotBuilder tools

- In a terminal, navigate to the samples folder (`BotBuilder-Samples\csharp_dotnetcore\14.NLP-With-Dispatch`) 

    ```bash
    cd BotBuilder-Samples\csharp_dotnetcore\14.NLP-With-Dispatch
    ```

- Install required tools - to successfully setup and configure all services this bot depend on, you need to install the MSBOT, LUIS, QnAMaker, Ludown, Dispatch CLI tools. 
    ```bash
    npm i -g msbot luis-apis qnamaker ludown botdispatch
    ```
- Configure required LUIS, QnA Maker and Dispatch services. See [here](#configure-services)

## Configure Services

This sample relies on [LUIS](https://luis.ai), [QnA Maker](https://qnamaker.ai) and [Dispatch](https://github.com/microsoft/botbuilder-tools//tree/master/packages/Dispatch) services. 

### Configure the LUIS service

To create required LUIS applications for this sample bot, 
- Create an account with [LUIS](https://luis.ai). If you already have an account, login to your account.
- Click on your name on top right corner of the screen -> settings and grab your authoring key.

To create the LUIS application this bot needs and update the .bot file configuration, in a terminal, 
- Clone this repository
- Navigate to BotBuilder-Samples\csharp_dotnetcore\14.NlpWithDispatch
- Run the following commands
```bash 
> ludown parse toluis --in Resources/homeautomation.lu -o CognitiveModels -n homeautomation.luis

> ludown parse toluis --in Resources/weather.lu -o CognitiveModels -n weather.luis

> luis import application --in CognitiveModels\homeautomation.luis --authoringKey <YOUR-LUIS-AUTHORING-KEY> --endpointBasePath https://westus.api.cognitive.microsoft.com/luis/api/v2.0 --msbot | msbot connect luis --stdin --name homeautomation.luis

> luis import application --in CognitiveModels\weather.luis --authoringKey <YOUR-LUIS-AUTHORING-KEY> --endpointBasePath https://westus.api.cognitive.microsoft.com/luis/api/v2.0 --msbot | msbot connect luis --stdin --name weather.luis
```

If you decide to change the names passed to msbot such as weather.luis, then you need to update the constants in [NlpDispatchBot.cs](NlpDispatch/NlpDispatchBot.cs). For example, if you change homeautomation.luis to just home, you would update the HomeAutomationLuisKey variable to "home" and the homeAutomationDispatchKey to the intent name assigned by dispatcher, which in this case will be "l_home".

Note: You can create the LUIS applications in one of the [LUIS authoring regions](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-reference-regions). 
You can use a different region, such as westus, westeurope or australiaeast via https://**LUIS-Authoring-Region**.api.cognitive.microsoft.com/luis/api/v2.0 in the commands above.

### Train and publish the LUIS models 
You need to train and publish the LUIS models that were created for this sample to work. You can do so using the following CLI commands

```bash
> msbot get service --name homeautomation.luis | luis train version --wait --stdin
> msbot get service --name homeautomation.luis | luis publish version --stdin
> msbot get service --name weather.luis | luis train version --wait --stdin
> msbot get service --name weather.luis | luis publish version --stdin
```

### Configure QnA Maker service
To create a new QnA Maker application for the bot, 
- Follow instructions [here](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) to create a new QnA Maker Azure resource.
- Navigate to your QnA Maker resource -> keys and copy the subscription key

To create the QnA Maker application and update the .bot file with the QnA Maker configuration,  
- Open a terminal
- Navigate to samples\14.NLP-With-Dispatch
- Run the following commands
```bash
> ludown parse toqna --in dialogs/qna/resources/sample-qna.lu -o cognitiveModels -n sample.qna

> qnamaker create kb --in cognitiveModels\sample.qna --subscriptionKey <YOUR-QNA-SUBSCRIPTION-KEY> --msbot | msbot connect qna --stdin --name sample.qna
```
### Train and publish the QnA Maker KB
You need to train and publish the QnA Maker Knowledge Bases that were created for this sample to work. You can do so using the following CLI commands

```bash
> msbot get service --name sample.qna | qnamaker publish kb --stdin
```

### Configure the Dispatch application
[Dispatch](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) is a CLI tool that enables you to create a dispatch NLP model across the different LUIS applications and / or QnA Maker Knowledge Bases you have for your bot. For this sample, you would have already created 2 LUIS applications (Home Automation and Weather) and one QnA Maker Knowledge base. 

To create a new dispatch model for these services and update the .bot file configuration, in a terminal:
- Navigate to samples\14.nlp-with-dispatch
- Run the following commands
```bash
> dispatch create -b nlp-with-dispatch.bot | msbot connect dispatch --stdin --name bot-dispatch
```
### Securing keys in your .bot file
Since your .bot file contains service Ids, subscription and authoring keys, its best to encrypt them. To encrypt the .bot file, run

```bash
msbot secret -n
```

This will generate a strong key, encrypt the bot file and print the key. Please keep this key securely.
Any time the bot file is encrypted, make sure to set the botFileSecret environment variable this sample relies on (either through the .env file or other means).

## Visual Studio
- Navigate to the samples folder (`BotBuilder-Samples\csharp_dotnetcore\14.NLP-With-Dispatch`) and open NLP-With-Dispatch-Bot.csproj in Visual Studio 
- Hit F5

## Visual Studio Code
- Open `BotBuilder-Samples\csharp_dotnetcore\14.NLP-With-Dispatch` sample folder.
- Bring up a terminal, navigate to BotBuilder-Samples\14.NLP-With-Dispatch folder
- type 'dotnet run'

## Testing the bot using Bot Framework Emulator
[Microsoft Bot Framework Emulator](https://github.com/microsoft/botframework-emulator) is a desktop application that allows bot developers to test and debug their bots on localhost or running remotely through a tunnel.

- Install the Bot Framework Emulator from [here](https://aka.ms/botframeworkemulator).

### Connect to bot using Bot Framework Emulator **V4**
- Launch Bot Framework Emulator
- File -> Open bot and navigate to `BotBuilder-Samples\csharp_dotnetcore\14.NlpWithDispatch` folder
- Select NLP-With-Dispatch-Bot.bot file

# Further reading

- [Azure Bot Service Introduction](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0)
- [QnA Maker Documentation](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview)
- [QnA Maker Command Line Tool](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker)
- [Application Insights](https://azure.microsoft.com/en-us/services/application-insights/)