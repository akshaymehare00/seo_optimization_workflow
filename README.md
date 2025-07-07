
# SEO Agent - Database to AI Optimization Workflow

[![n8n](https://img.shields.io/badge/n8n-workflow-red)](https://n8n.io)
[![AI](https://img.shields.io/badge/AI-OpenAI%20%7C%20HuggingFace%20%7C%20Ollama-blue)](https://github.com)
[![Database](https://img.shields.io/badge/Database-PostgreSQL-blue)](https://postgresql.org)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

An intelligent n8n workflow that automatically generates SEO-optimized titles and meta descriptions for your database content using multiple AI providers.

## 🚀 Features

- **Multi-AI Provider Support**: Works with OpenAI, Hugging Face, and Ollama (local)
- **Database Integration**: Direct PostgreSQL integration for seamless data processing
- **Webhook API**: RESTful API endpoint for easy integration
- **Intelligent Fallback**: Robust error handling and AI response parsing
- **Flexible Content Processing**: Handles various content fields (title, description, content, name)
- **Real-time Updates**: Immediately updates your database with optimized SEO data

## 🏗️ Architecture

```
Webhook Trigger → Input Validation → Database Fetch → AI Processing → Database Update → Response
```

### Workflow Components

1. **Webhook Trigger**: Accepts POST requests with record ID and table name
2. **Input Validation**: Ensures required parameters are provided
3. **Database Fetch**: Retrieves content from PostgreSQL
4. **AI Processing**: Parallel processing with multiple AI providers
5. **Response Parser**: Intelligently extracts SEO data from AI responses
6. **Database Update**: Updates records with optimized SEO fields
7. **API Response**: Returns structured success/error responses

## 📋 Prerequisites

- n8n instance (self-hosted or cloud)
- PostgreSQL database
- At least one AI provider:
  - OpenAI API key
  - Hugging Face API key
  - Ollama (local installation)

## 🛠️ Setup Instructions

### 1. Database Setup

Ensure your PostgreSQL tables have the following columns:
```sql
-- Add these columns to your existing tables
ALTER TABLE your_table_name ADD COLUMN seo_title VARCHAR(255);
ALTER TABLE your_table_name ADD COLUMN seo_meta_description TEXT;
ALTER TABLE your_table_name ADD COLUMN seo_updated_at TIMESTAMP;
```

### 2. n8n Configuration

1. Import the workflow JSON file into your n8n instance
2. Configure credentials:
   - **PostgreSQL**: Database connection details
   - **OpenAI**: API key (optional)
   - **Hugging Face**: API token (optional)
3. Update the webhook URL in your application

### 3. AI Provider Setup

#### OpenAI (Recommended)
```bash
# Set your OpenAI API key in n8n credentials
API_KEY=your_openai_api_key
```

#### Hugging Face
```bash
# Set your Hugging Face token
HF_TOKEN=your_huggingface_token
```

#### Ollama (Local)
```bash
# Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# Pull the model
ollama pull llama2
```

## 🔧 Usage

### API Endpoint

**POST** `/webhook/seo-optimize`

#### Request Body
```json
{
  "id": "123",
  "table": "products",
  "content_field": "description"
}
```

#### Parameters
- `id` (required): Record ID to optimize
- `table` (required): Database table name
- `content_field` (optional): Field containing content (defaults to 'content')

#### Success Response
```json
{
  "success": true,
  "id": "123",
  "table": "products",
  "updated_fields": {
    "seo_title": "Premium Quality Product - Best Choice 2024",
    "seo_meta_description": "Discover our premium quality product with exceptional features. Perfect for professionals seeking reliable solutions. Order now with free shipping!"
  },
  "timestamp": "2025-07-07T10:30:00Z"
}
```

#### Error Response
```json
{
  "success": false,
  "error": "Record not found",
  "message": "No record found with ID 123 in table products",
  "timestamp": "2025-07-07T10:30:00Z"
}
```

### Integration Examples

#### PHP/Laravel
```php
$response = Http::post('https://your-n8n-instance.com/webhook/seo-optimize', [
    'id' => $product->id,
    'table' => 'products',
    'content_field' => 'description'
]);

if ($response->json()['success']) {
    // SEO data updated successfully
    $product->refresh();
}
```

#### Node.js/Express
```javascript
const response = await fetch('https://your-n8n-instance.com/webhook/seo-optimize', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        id: productId,
        table: 'products',
        content_field: 'description'
    })
});

const result = await response.json();
console.log(result);
```

#### Python/Django
```python
import requests

response = requests.post('https://your-n8n-instance.com/webhook/seo-optimize', json={
    'id': product.id,
    'table': 'products',
    'content_field': 'description'
})

if response.json()['success']:
    # Handle successful optimization
    product.refresh_from_db()
```

## 🎯 SEO Optimization Details

The AI generates SEO content following these guidelines:

### Title Optimization
- Maximum 60 characters
- Keyword-rich and compelling
- Focused on search intent
- Click-worthy and engaging

### Meta Description Optimization
- 150-160 characters optimal length
- Actionable and descriptive
- Includes relevant keywords
- Encourages click-through

### AI Prompt Engineering
The workflow uses carefully crafted prompts to ensure:
- Consistent JSON output format
- SEO best practices compliance
- Brand voice consideration
- Search intent alignment

## 🔧 Customization

### Modify AI Prompt
Edit the `Prepare AI Input` node to customize the AI prompt:

```javascript
const prompt = `Your custom SEO prompt here...
Content: ${content}
Current Title: ${currentTitle}

Requirements:
- Your specific requirements
- Custom formatting rules
- Brand guidelines

Return JSON: {"seo_title": "...", "seo_meta_description": "..."}`;
```

### Add Custom Fields
Extend the database query in `Fetch Data from PostgreSQL`:

```sql
SELECT id, content, title, name, description,
       seo_title, seo_meta_description,
       custom_field1, custom_field2
FROM {{ $json.table }}
WHERE id = {{ $json.id }}
```

### Configure AI Models
Update model parameters in AI processing nodes:

```javascript
// OpenAI
"chatId": "gpt-4", // Change model
"temperature": 0.7, // Adjust creativity
"maxTokens": 200    // Control response length

// Ollama
"model": "llama2:13b" // Use different model size
```

## 🚦 Error Handling

The workflow includes comprehensive error handling:

- **Input Validation**: Ensures required parameters
- **Database Errors**: Handles connection and query issues
- **AI Processing**: Manages API failures and timeouts
- **Response Parsing**: Fallback for malformed AI responses
- **HTTP Responses**: Proper status codes and error messages

## 📊 Monitoring & Logging

### Execution Logs
Monitor workflow executions in n8n:
- Success/failure rates
- Processing times
- Error patterns
- AI provider performance

### Database Tracking
Track SEO optimization history:
```sql
-- Query recently optimized records
SELECT id, title, seo_title, seo_updated_at
FROM your_table
WHERE seo_updated_at > NOW() - INTERVAL '24 hours'
ORDER BY seo_updated_at DESC;
```

## 🔒 Security Considerations

- Validate input parameters to prevent SQL injection
- Use parameterized queries
- Implement rate limiting on webhook endpoint
- Secure API credentials in n8n credential store
- Monitor AI API usage and costs

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Test your changes thoroughly
4. Submit a pull request with detailed description

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- **Issues**: Report bugs and feature requests
- **Discussions**: Ask questions and share experiences
- **Wiki**: Detailed documentation and tutorials

## 🔗 Related Projects

- [n8n](https://n8n.io) - Workflow automation platform
- [OpenAI API](https://openai.com/api) - AI language models
- [Ollama](https://ollama.ai) - Local AI model runner

---

**Made with ❤️ by the community**

*Star this repository if you find it useful!*