from flask import Flask, request
from twilio.twiml.voice_response import VoiceResponse, Gather
import os

app = Flask(__name__)

@app.route('/')
def home():
    return '''
    <html>
        <head>
            <title>VoiceFlow AI - LIVE!</title>
            <style>
                body { 
                    font-family: Arial; 
                    text-align: center;
                    padding: 50px;
                    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                }
                .card {
                    background: white;
                    padding: 40px;
                    border-radius: 15px;
                    max-width: 600px;
                    margin: 0 auto;
                }
                h1 { color: #667eea; }
                .success { 
                    background: #10b981; 
                    color: white; 
                    padding: 20px; 
                    border-radius: 10px;
                    margin: 20px 0;
                    font-size: 24px;
                }
            </style>
        </head>
        <body>
            <div class="card">
                <h1>🎉 VoiceFlow AI is LIVE!</h1>
                <div class="success">✅ Your System is Working!</div>
                <h2>System Status:</h2>
                <p>✅ Server Running</p>
                <p>✅ Ready to Accept Calls</p>
                <h3>Call your Twilio number to test!</h3>
            </div>
        </body>
    </html>
    '''

@app.route('/voice/incoming', methods=['POST'])
def incoming_call():
    response = VoiceResponse()
    response.say("Hello! Thank you for calling VoiceFlow AI. This is your AI assistant speaking. How can I help you today?", voice='Polly.Joanna')
    
    gather = Gather(
        input='speech',
        action='/voice/process',
        method='POST',
        speechTimeout='auto'
    )
    response.append(gather)
    
    return str(response)

@app.route('/voice/process', methods=['POST'])
def process_speech():
    speech = request.values.get('SpeechResult', '')
    
    response = VoiceResponse()
    response.say(f"I heard you say: {speech}. Thank you for testing VoiceFlow AI. This system is working perfectly. Goodbye!", voice='Polly.Joanna')
    response.hangup()
    
    return str(response)

@app.route('/health')
def health():
    return {'status': 'healthy', 'message': 'VoiceFlow AI is running!'}

if __name__ == '__main__':
    port = int(os.getenv('PORT', 8080))
    app.run(host='0.0.0.0', port=port)
- **Custom Build Command**: This is the command that will be ran to build your final application. Railpack will find the best command for this, usually `npm run build` for JS-based projects, `cargo build --release` for Rust projects, and more. If your application needs to trigger something else to build your project, customize that command here.
- [**Pre-Deploy Command**](https://docs.railway.com/guides/pre-deploy-command): These are one or more commands that will be ran before running the main start command. A common use for this is database migrations. If your application needs to run a command before starting your main application, put that in a Pre-Deploy Command.
- **Custom Start Command**: This is the command that will actually run your application. Defaulted as `npm run start` for JS-based applications. If you need to start your application in a way that's different than expected, change that here.

### Deploy Options

Some default options are applied when your application is deployed on Railway. They can change where your deployments are ran, how they are ran, and what happens to them while they're running. Here are some things you can change:

- [**Replicas**](https://docs.railway.com/reference/scaling#horizontal-scaling-with-replicas): By default, your deployment will go out with a single instance in your [preferred region](https://docs.railway.com/guides/optimize-performance#set-a-preferred-region). With replicas, you can deploy multiple instances of your application in one or more regions. The Railway network will load balance incoming requests between the available replicas and serve the ones closest to your users.
- **Deployment Region**: If you haven't configured any Replicas, your services will be deployed to the [preferred region](https://docs.railway.com/guides/optimize-performance#set-a-preferred-region) configured in your workspace. To change an individual service's region, go to Settings > Deploy > Regions.
- **Scheduled Executions**: If your deployment isn't a long-running service, like most web or backend services are, your deployment will be run once then exit. If the service is intended to be a repeated task, you can create a cron schedule under Settings > Deploy that will re-run your deployment according to the schedule. Each time it runs and exits successfully, it'll be marked as Completed.
- [**Serverless Deployments**](https://docs.railway.com/reference/app-sleeping): By default, services are long-running and will continue to run unless they error. You can configure your services to pause if no traffic is going to them, and start it back up once a request comes in. Sleeping services don't incur resource usage. While your service is starting back up, the requests will be queued and delivered to the service once its fully up and running.
- [**Healthchecks**](https://docs.railway.com/reference/healthchecks): By default, as soon as your deployment is launched, it's considered "healthy". You can configure an endpoint path under Deploy > Healthcheck Path that Railway will fetch once it deploys your service to ensure that your service actually started up before it removes the previous deployment. This lets Railway know when your services are fully healthy and ready to receive traffic, so it doesn't start serving requests to a service that hasn't fully started yet.

## Networking

Constructing efficient networking setups yourself can be tricky and time-consuming to get right. Included in the Railway experience is fast networking that just works and can help you build more modular services.

### Private Networking

[Private Networking](https://docs.railway.com/guides/private-networking) is a feature that lets your services communicate to other running services within your project simply by its service name. Under Settings > Networking, you can configure the specific domain that is given to your service. You can provide this private networking domain name to your services with [Reference Variables](https://docs.railway.com/guides/variables#reference-variables).

Private Networking domains are available with [Railway-provided Variables](https://docs.railway.com/guides/variables#railway-provided-variables) that you can [provide to your other services](https://docs.railway.com/guides/variables#referencing-a-shared-variable), elimiating the need to hard-code their values.

Under the hood, Railway connects your services together with a WireGuard mesh and a DNS resolver that is scoped to your project and environment. Services running inside of one project or environment aren't able to reach the services running in a different project. This also applies to environments within a project being unable to reach other environments.

### Railway-provided Domains

Railway provides a [customizable, Railway-branded domain](https://docs.railway.com/guides/public-networking#railway-provided-domain) for your service that you can connect to a port running on your service and instantly make it available to the public internet. These domains require no DNS configuration on your end, and are perfect if you don't have a custom domain of your own just yet but still want to access your application.

### Custom Domains

If you have a [custom domain](https://docs.railway.com/guides/public-networking#custom-domains) that you've bought from a domain registrar, you can connect this to your Railway service to serve your application under. Railway will walk you through the configuration of the required DNS records, alert you if the configuration is wrong, and automatically provision SSL certificates for you so you can serve your application with `https://`.

## Integrations

Integrations let you interact with your Railway workspace outside of the Railway dashboard.

### CLI

The Railway CLI lets you interact with your Railway project from your local machine on the command line, allowing you to do things like:

- Triggering deployments programmatically
- Run your services locally using configured environment variables from your Railway project
- Launch new Railway projects from the terminal
- Deploy <a href="https://docs.railway.com/reference/templates" target="_blank">templates</a>

### Public API

The Railway <a href="https://docs.railway.com/guides/public-api" target="_blank">public API</a> is built with GraphQL and is the same API that powers the Railway dashboard. Similar to the CLI, you can interact with your Railway project programmatically by communicating with the API.

### Webhooks

Projects can also configure [Webhook URLs](https://docs.railway.com/guides/webhooks) that can receive a number of events that are triggered by your services and other things inside your project. Using these in tandem with the Railway API can make some powerful workflows that respond to the rest of your project. They can also be used to deliver critical alerts to your team's Slack workspaces.

## Environments

Railway environments are isolated instances of the services running in your production environment that let you iterate on your workspace without affecting your production workloads. You can:

- Have development environments for each team member that mirror your production environment
- Have separate staging and production environments

Within a service and environment, you can specify which branch of your GitHub repository to deploy to that environment when a commit is pushed.

## Observability

Railway services and deployments will likely output logs to describe how they are being built, what they are doing while running, or errors during either of these.

These logs, when sent to the standard output or standard error (like when using `console.log()`, `print()`, and `console.error()`) will be shown in the [Build and Deploy tabs](https://docs.railway.com/guides/logs#build--deploy-panel) (depending on where they originated) of each service and can be viewed in the [Observability](https://docs.railway.com/guides/logs#log-explorer) page of your project, from which you can view logs from all your services.
