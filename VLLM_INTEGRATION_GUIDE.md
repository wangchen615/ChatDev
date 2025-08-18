# vLLM Integration Guide for ChatDev

This guide explains how to integrate locally hosted vLLM endpoints with ChatDev, replacing OpenAI API calls.

## 🚀 **Quick Start**

### **1. Set Environment Variables**
```bash
# Set vLLM endpoint (adjust URL and port as needed)
export VLLM_ENDPOINT="http://localhost:8000/v1"
export VLLM_MODEL_NAME="default"

# Optional: Keep OpenAI API key for fallback
export OPENAI_API_KEY="your_openai_api_key_here"
```

### **2. Run ChatDev with vLLM**
```bash
# Use vLLM model for any task
python3 run.py --task "create a hello world program" --name "HelloWorld" --model VLLM

# Use vLLM for coding tasks
python3 run.py --task "create a calculator" --name "Calculator" --model VLLM

# Use vLLM for general tasks
python3 run.py --task "create a todo app" --name "TodoApp" --model VLLM
```

## 🔧 **What Was Changed**

### **1. Model Type Definitions (`camel/typing.py`)**
- Added new vLLM model type:
  - `VLLM` - Local vLLM model (model name controlled by environment variable)

### **2. Model Backend (`camel/model_backend.py`)**
- Created `VLLMModel` class that implements the `ModelBackend` interface
- Handles HTTP requests to local vLLM endpoints
- Converts vLLM responses to OpenAI-compatible format
- Maintains token counting and logging

### **3. Model Factory (`camel/model_backend.py`)**
- Updated `ModelFactory.create()` to route vLLM model types to `VLLMModel`
- Maintains backward compatibility with OpenAI models

### **4. Run Script (`run.py`)**
- Added vLLM model options to command line arguments
- Updated help text to include vLLM options

### **5. Dependencies (`requirements.txt`)**
- Added `requests>=2.31.0` for HTTP communication with vLLM

## 🌐 **vLLM Endpoint Requirements**

Your vLLM endpoint must support the OpenAI-compatible API format:

### **Endpoint**: `POST /v1/chat/completions`

### **Request Format**:
```json
{
  "model": "your-model-name",
  "messages": [
    {"role": "user", "content": "Hello, how are you?"}
  ],
  "temperature": 0.2,
  "top_p": 1.0,
  "max_tokens": 4096,
  "stream": false
}
```

### **Response Format**:
```json
{
  "id": "response-id",
  "object": "chat.completion",
  "created": 1234567890,
  "model": "your-model-name",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "I'm doing well, thank you!"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 8,
    "total_tokens": 18
  }
}
```

## 🚀 **Starting vLLM Server**

### **Option 1: Using vLLM CLI**
```bash
# Install vLLM
pip install vllm

# Start server with a model
vllm serve --model meta-llama/Llama-2-7b-chat-hf --port 8000
```

### **Option 2: Using Docker**
```bash
# Pull vLLM Docker image
docker pull vllm/vllm-openai

# Run vLLM server
docker run --gpus all -p 8000:8000 vllm/vllm-openai --model meta-llama/Llama-2-7b-chat-hf
```

### **Option 3: Using Hugging Face TGI**
```bash
# Install text-generation-inference
pip install text-generation-inference

# Start server
text-generation-launcher --model-id meta-llama/Llama-2-7b-chat-hf --port 8000
```

## 🔍 **Testing vLLM Integration**

### **1. Test Endpoint**
```bash
curl -X POST "http://localhost:8000/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "default",
    "messages": [{"role": "user", "content": "Hello!"}],
    "temperature": 0.2
  }'
```

### **2. Test ChatDev Integration**
```bash
# Simple test
python3 run.py --task "print hello world" --name "Test" --model VLLM_LOCAL
```

## 🚨 **Troubleshooting**

### **Common Issues**

#### **1. Connection Refused**
```bash
# Check if vLLM server is running
curl http://localhost:8000/v1/models

# Check server logs
# Look for port conflicts or GPU issues
```

#### **2. Model Not Found**
```bash
# Check available models
curl http://localhost:8000/v1/models

# Verify model name in environment variable
echo $VLLM_MODEL_NAME
```

#### **3. Response Format Errors**
- Ensure vLLM server returns OpenAI-compatible format
- Check server logs for errors
- Verify model supports chat completions

#### **4. Performance Issues**
- Use appropriate model size for your hardware
- Consider using quantized models (e.g., AWQ, GPTQ)
- Monitor GPU memory usage

### **Debug Mode**
Set environment variable for detailed logging:
```bash
export VLLM_DEBUG=1
```

## 📊 **Performance Comparison**

| Metric | OpenAI GPT-3.5 | vLLM Local | Notes |
|--------|----------------|------------|-------|
| **Latency** | 100-500ms | 50-200ms | Depends on model size |
| **Cost** | $0.002/1K tokens | $0.00 | Local hosting |
| **Reliability** | 99.9% | 95-99% | Depends on hardware |
| **Customization** | Limited | Full | Model fine-tuning possible |

## 🔄 **Fallback Strategy**

If vLLM fails, ChatDev can fall back to OpenAI:

```bash
# Set both endpoints
export VLLM_ENDPOINT="http://localhost:8000/v1"
export OPENAI_API_KEY="your_key_here"

# ChatDev will try vLLM first, then OpenAI if needed
```

## 🎯 **Next Steps**

1. **Test with your specific model**
2. **Optimize model parameters**
3. **Consider model fine-tuning for coding tasks**
4. **Monitor performance and adjust accordingly**
5. **Set up monitoring and logging**

## 📚 **Additional Resources**

- [vLLM Documentation](https://docs.vllm.ai/)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
- [ChatDev Documentation](https://github.com/OpenBMB/ChatDev)
- [Model Hosting Guide](https://docs.vllm.ai/en/latest/deployment/)

## 🤝 **Support**

If you encounter issues:
1. Check vLLM server logs
2. Verify endpoint configuration
3. Test with simple curl commands
4. Check ChatDev logs for detailed error messages
