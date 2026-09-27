# Building a Secure AWS API Gateway & Lambda Endpoint with Native API Key Authorization

This repository provides a step-by-step guide to building a REST API using **AWS API Gateway** and **AWS Lambda** (Python 3.12), secured using **native API Gateway Keys and Usage Plans**. 

In this example, we build a `/combine` endpoint that accepts two string values (`str1` and `str2`) via a JSON payload and returns the concatenated string.

---

## 🏗 Architecture Overview

```
[ Client / Web App ] 
         │  (HTTP POST with 'x-api-key' & JSON payload)
         ▼
[ AWS API Gateway (TestAPI1) ] 
         │  (Validates API Key via Usage Plan)
         ▼
[ AWS Lambda (CombineStringsFunction) ] 
         │  (Parses JSON & concatenates strings)
         ▼
[ JSON Response ]
```

---

## 🛠 Step 1: Create the Backend Lambda Function

1. Open the **AWS Lambda Console** and click **Create function**.
2. Name the function `CombineStringsFunction` and select **Python 3.12** (or 3.11).
3. Replace the contents of `lambda_function.py` with the following code:

```python
import json

def lambda_handler(event, context):
    """
    Lambda function that receives a JSON payload with 'str1' and 'str2'
    and returns their concatenated result.
    """
    try:
        # Extract and parse the raw JSON string passed from API Gateway
        raw_body = event.get('body') or '{}'
        body = json.loads(raw_body)
    except Exception:
        body = {}
    
    # Read the two input strings (defaulting to empty strings if missing)
    string1 = body.get('str1', '')
    string2 = body.get('str2', '')
    
    # Combine the strings
    combined_result = string1 + string2
    
    # Return standard HTTP 200 response with JSON output
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'  # Ensures CORS support
        },
        'body': json.dumps({
            'string1': string1,
            'string2': string2,
            'result': combined_result
        })
    }
```

4. Click **Deploy**.

---

## 🌐 Step 2: Create the REST API in API Gateway

1. Open the **AWS API Gateway Console**.
2. Choose **REST API** (not Private or HTTP API) and click **Build**.
3. Fill in the initial settings:
   * **Choose the protocol:** `REST`
   * **Create new API:** `New API`
   * **API name:** `TestAPI1`
   * **Endpoint Type:** `Regional`
4. Click **Create API**.

---

## 🔀 Step 3: Create Resource and POST Method

1. Under **Resources**, click **Create resource**.
   * **Resource name:** `combine`
   * Enable **CORS (Cross-Origin Resource Sharing)**.
   * Click **Create resource**.
2. Select the `/combine` resource and click **Create method**:
   * **Method type:** `POST`
   * **Integration type:** `Lambda function`
   * Enable **Lambda proxy integration** *(Crucial for passing `event['body']` directly)*.
   * **Lambda function:** Select `CombineStringsFunction`.
   * Click **Create method**.

---

## 🔑 Step 4: Configure Native API Key Authorization

### 4.1 Enforce API Key on the Method
1. Click on the **POST** method under `/combine`.
2. Go to the **Method Request** tab and click **Edit** under *Method Request settings*.
3. Set **Authorization** to `None`.
4. Set **API Key Required** to `true`.
5. Click **Save**.

### 4.2 Create the API Key
1. In the left navigation menu, go to **API Keys**.
2. Click **Create API key**.
3. Set Name to `MyMainKey` (Select **Auto Generate**).
4. Click **Save** and make note of the generated key string.

### 4.3 Link Key to a Usage Plan
1. In the left menu, select **Usage Plans** and click **Create usage plan**.
2. Name it `BasicPlan` and configure optional throttling/quota settings.
3. Under **Associated API stages**, click **Add API stage**:
   * Select `TestAPI1` and choose your stage (e.g., `prod`).
4. Under **Associated API keys**, click **Add API key**:
   * Select `MyMainKey`.
5. Click **Save**.

---

## 🚀 Step 5: Deploy the API

1. Go back to **TestAPI1 > Resources**.
2. Click **Deploy API** at the top right.
3. Select **Stage**: `*New Stage*` (e.g., `prod`).
4. Click **Deploy**.
5. Copy the **Invoke URL** displayed at the top of the Stage dashboard.

---

## 🧪 Testing the Endpoint

### ❌ Unauthenticated Request (Expect `403 Forbidden`)

Making a request without the `x-api-key` header will be rejected by API Gateway before triggering Lambda:

```bash
curl -X POST "https://<YOUR-API-ID>.execute-api.<REGION>.amazonaws.com/prod/combine" \
     -H "Content-Type: application/json" \
     -d '{
       "str1": "Hello",
       "str2": "World"
     }'
```

**Response:**
```json
{
  "message": "Forbidden"
}
```

---

### ✅ Authenticated Request (Expect `200 OK`)

Pass your generated API key inside the `x-api-key` header:

```bash
curl -X POST "https://<YOUR-API-ID>.execute-api.<REGION>.amazonaws.com/prod/combine" \
     -H "x-api-key: YOUR_GENERATED_API_KEY_HERE" \
     -H "Content-Type: application/json" \
     -d '{
       "str1": "Hello",
       "str2": "World"
     }'
```

**Response:**
```json
{
  "string1": "Hello",
  "string2": "World",
  "result": "HelloWorld"
}
```

---

## 💡 Key Learnings & Pitfalls

1. **Lambda Proxy Integration:** Enabling proxy integration ensures that headers, path parameters, and request bodies are wrapped inside the `event` object (`event['body']`).
2. **Usage Plan Linking:** An API Key in API Gateway will **not** grant access until it is explicitly bound to both an API Stage AND a Usage Plan.
3. **CORS & Preflight Requests:** When enabling CORS, API Gateway generates an `OPTIONS` method. Preflight requests do not pass API keys, so `OPTIONS` methods must **never** have *API Key Required* set to `true`.
