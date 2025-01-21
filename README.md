# Building-a-News-Grabber-Chatbot-using-IBM-Watson-HBot-with-Discovery-NLU-and-Knowledge-Studio-
Building a News Grabber Chatbot using IBM Watson (TJBot with Discovery, NLU, and Knowledge Studio)

IBM Watson offers a suite of powerful AI services that can be used to build chatbots capable of processing, understanding, and interacting with text data. In this guide, we will build a News Grabber Chatbot using three major Watson services:

    Watson Discovery: To access news articles and extract valuable information.
    Watson Natural Language Understanding (NLU): To analyze and understand text and extract insights.
    Watson Knowledge Studio: To customize Watson NLU for specific domains or intents.
    TJBot: A physical robot that can interface with Watson services.

We will create a simple chatbot that grabs the latest news articles, processes the text to extract key insights, and responds to queries using the information it retrieves.
1. Prerequisites

    IBM Watson Services: You need to set up and configure Watson Discovery, Watson NLU, and Watson Knowledge Studio from the IBM Cloud console.
        Create a Watson Discovery service: You'll need the API key and endpoint to connect to the service.
        Create a Watson NLU service: Get the API key and endpoint to connect to NLU.
        Create a Watson Knowledge Studio service: This is used to train and customize the NLU model to understand domain-specific intents and entities.

    Set up TJBot: TJBot is a programmable robot powered by IBM Watson, typically using a Raspberry Pi. Follow the TJBot setup guide for hardware setup and installing the required dependencies.

2. Install Required Python Libraries

You’ll need the following Python libraries to interact with Watson services and TJBot:

pip install ibm-watson
pip install flask
pip install requests

    ibm-watson: SDK to interact with Watson services.
    flask: To build a simple web server for your chatbot.
    requests: To fetch news articles from a news API.

3. Setting Up Watson Discovery

To use Watson Discovery to grab news articles, you'll need an API key and endpoint. You can create a collection in Watson Discovery that will serve as the source of the news articles. You can either use your own dataset or use a news API.

Here’s how to integrate Watson Discovery into your bot:

from ibm_watson import DiscoveryV1
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator

# Watson Discovery Setup
authenticator = IAMAuthenticator('your-discovery-api-key')
discovery = DiscoveryV1(version='2021-08-01', authenticator=authenticator)
discovery.set_service_url('your-discovery-endpoint')

# Fetching latest news from a collection
def fetch_latest_news():
    query = 'news'
    query_params = {
        'query': query,
        'count': 5,  # Get top 5 articles
        'highlight': True
    }

    # Perform a query on the collection
    response = discovery.query('your-collection-id', **query_params).get_result()
    articles = response['results']
    
    # Extract and format the articles
    news_articles = []
    for article in articles:
        news_articles.append({
            'title': article['title'],
            'url': article['url'],
            'snippet': article['text'][:200]  # Get snippet of the article
        })
    
    return news_articles

In this code, the fetch_latest_news() function performs a query on your Watson Discovery collection and returns the titles, URLs, and snippets of the top 5 news articles.
4. Setting Up Watson NLU (Natural Language Understanding)

Watson NLU will process user input and extract entities, categories, keywords, and sentiment. For instance, if a user asks a question like "What is the latest news on technology?", the NLU service will analyze the input and provide relevant insights.

from ibm_watson import NaturalLanguageUnderstandingV1
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator

# Watson NLU Setup
nlu_authenticator = IAMAuthenticator('your-nlu-api-key')
nlu = NaturalLanguageUnderstandingV1(
    version='2021-08-01',
    authenticator=nlu_authenticator
)
nlu.set_service_url('your-nlu-endpoint')

# Analyze user query for entities, categories, and sentiment
def analyze_query(user_query):
    response = nlu.analyze(
        text=user_query,
        features={
            'entities': {},
            'keywords': {},
            'sentiment': {},
            'categories': {}
        }
    ).get_result()
    
    return response

In this function, the analyze_query() method processes the user's input to extract entities, keywords, sentiment, and categories.
5. Setting Up Watson Knowledge Studio (Optional)

Watson Knowledge Studio is used to train the NLU model with domain-specific data, such as identifying industry-specific entities or intents. You can upload training data and build a custom model for your chatbot.

You can use the watson-knowledge-studio library to interact with this service, but setting it up typically involves training and deploying a custom NLU model. You would upload your labeled training data to Knowledge Studio, train the model, and deploy it to use in your chatbot.

For now, let’s assume that we are using Watson NLU's pre-trained model for general understanding.
6. Building the Flask Web Server

Now, let’s create a simple web server using Flask to interact with the user and TJBot. The server will:

    Take user input via HTTP requests.
    Analyze the query using Watson NLU.
    Fetch the latest news from Watson Discovery.
    Use TJBot to provide a physical response (e.g., text-to-speech or LEDs).

from flask import Flask, request, jsonify
import json
import TJBot

# Initialize TJBot (Assuming the robot is set up and connected)
my_tjbot = TJBot.TJBot()

# Initialize Flask App
app = Flask(__name__)

@app.route('/ask', methods=['POST'])
def ask():
    user_query = request.json['query']
    
    # Step 1: Analyze query with NLU
    nlu_response = analyze_query(user_query)
    
    # Step 2: Fetch latest news
    news_articles = fetch_latest_news()
    
    # Step 3: Generate response based on NLU and Discovery results
    if 'news' in user_query.lower():
        # Step 4: Respond with the top 3 news articles
        news_response = "\n".join([f"{article['title']} - {article['url']}" for article in news_articles[:3]])
        response = {"answer": f"Here are the latest news articles:\n{news_response}"}
        
        # Use TJBot to speak the response
        my_tjbot.say(response['answer'])
    else:
        response = {"answer": "Sorry, I didn't understand. Could you rephrase?"}
        my_tjbot.say(response['answer'])
    
    return jsonify(response)

if __name__ == '__main__':
    app.run(debug=True)

7. Running the Flask Server

To run the Flask server, execute the script:

python chatbot_server.py

This will start the server on http://localhost:5000. You can send a POST request to /ask with the following JSON payload to interact with the bot:

{
    "query": "What's the latest news on technology?"
}

8. Interacting with TJBot

Once the user sends a request, the bot:

    Analyzes the query using Watson NLU.
    Fetches the latest news articles from Watson Discovery.
    Returns a list of news articles (titles, URLs) and uses TJBot's text-to-speech to speak the response.

9. Example Request and Response

    Request: User sends a POST request with a query like "What is the latest news on technology?".

    Response:
        The bot analyzes the query for entities and sentiment.
        It fetches the latest news articles from Watson Discovery related to the query.
        It responds with the top 3 relevant articles and reads them aloud using TJBot's speech synthesis.

Conclusion

In this tutorial, we demonstrated how to build a News Grabber Chatbot using IBM Watson services such as Discovery, Natural Language Understanding (NLU), and optionally Knowledge Studio. The chatbot can process user queries to fetch news articles, understand intent and entities via NLU, and provide responses using TJBot, a physical robot.

This setup enables creating intelligent, conversational bots that can interact with users in real-time, leveraging Watson’s AI capabilities to deliver personalized and relevant responses.
