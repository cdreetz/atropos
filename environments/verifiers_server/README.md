# Verifiers Environment Integration

A comprehensive integration layer that connects PrimeIntellect's [Verifiers](https://github.com/PrimeIntellect-ai/verifiers) environment ecosystem with Atropos' RL training framework. This environment enables seamless training on any environment from the PrimeIntellect Environments Hub.

## 🎯 Overview

The Verifiers Environment Integration provides a bridge between:
- **PrimeIntellect's Verifiers Library**: A comprehensive ecosystem of RL environments for LLM training
- **Atropos Framework**: Nous Research's async RL training infrastructure

This integration allows you to leverage hundreds of pre-built environments from the Environments Hub or create your own custom environments using the Verifiers framework.

## ✨ Key Features

### 🔌 **Seamless Integration**
- Direct integration with PrimeIntellect's Verifiers package
- Automatic rubric and reward function loading
- Native support for multi-turn interactions
- Compatible with all Verifiers-based environments

### 🎮 **Environment Flexibility**
- Access to the entire Environments Hub catalog
- Support for custom environment configurations
- Dynamic reward function composition
- Configurable environment arguments

### 📊 **Comprehensive Metrics**
- Automatic score calculation with weighted rewards
- Built-in evaluation support
- WandB integration for tracking
- Detailed performance monitoring

### 🔧 **Developer-Friendly**
- Simple environment loading via `vf.load_environment()`
- Automatic parser and rubric extraction
- Flexible reward weighting system
- Easy-to-extend base implementation

## 📋 Prerequisites

### Installation

1. **Install Prime CLI** (for environment management):
```bash
uv tool install prime
```

2. **Login to Prime**:
```bash
prime login
```

3. **Install a Verifiers Environment**:
```bash
# Example: Install the Wordle environment
prime env install will/wordle

# Or install any other environment from the Hub
prime env install owner/environment-name
```

4. **Install Verifiers Package**:
```bash
pip install "verifiers>=0.1.5.post0"
```

## 🚀 Quick Start

### Basic Usage

```python
from environments.verifiers_server import VerifiersEnv, VfEnvConfig
from atroposlib.envs.base import APIServerConfig
import os

# Create configuration
env_config = VfEnvConfig(
    vf_env_name="wordle",  # Name of the installed environment
    env_args={},           # Optional environment-specific arguments
    group_size=8,
    use_wandb=True,
    rollout_server_url="http://localhost:8000",
    total_steps=1000,
    batch_size=12,
    steps_per_eval=100,
    max_token_length=2048,
    wandb_name="verifiers-wordle"
)

# Configure API server
server_configs = [
    APIServerConfig(
        model_name="gpt-4o-mini",
        base_url=None,
        api_key=os.getenv("OPENAI_API_KEY"),
        num_requests_for_eval=256,
    ),
]

# Initialize environment
env = VerifiersEnv(
    config=env_config,
    server_configs=server_configs,
)

# Setup and run
await env.setup()
metrics = await env.evaluate()
```

### CLI Usage

```bash
# Start the API server
run-api

# In another terminal, serve the environment
python environments/verifiers_server.py serve \
    --env.vf_env_name wordle \
    --openai.model_name gpt-4o-mini \
    --openai.api_key $OPENAI_API_KEY

# Process mode for testing
python environments/verifiers_server.py process \
    --env.vf_env_name wordle \
    --env.data_path_to_save_groups wordle_rollouts.jsonl

# Evaluate mode
python environments/verifiers_server.py evaluate \
    --env.vf_env_name wordle \
    --openai.model_name gpt-4o-mini
```

## ⚙️ Configuration

### VfEnvConfig Options

```python
class VfEnvConfig(BaseEnvConfig):
    vf_env_name: str = ""        # Name of the Verifiers environment
    env_args: dict = {}           # Environment-specific arguments

    # Standard Atropos options
    group_size: int = 8
    use_wandb: bool = True
    rollout_server_url: str = "http://localhost:8000"
    total_steps: int = 1000
    batch_size: int = 12
    steps_per_eval: int = 100
    max_token_length: int = 2048
    wandb_name: str = "verifiers-env"
```

### Environment-Specific Arguments

Different Verifiers environments may accept different arguments. Check the specific environment's documentation:

```python
# Example: Custom environment arguments
env_config = VfEnvConfig(
    vf_env_name="my-custom-env",
    env_args={
        "difficulty": "hard",
        "num_rounds": 10,
        "custom_param": "value"
    }
)
```

## 🎮 Available Environments

The Verifiers integration supports any environment from the PrimeIntellect Environments Hub. Popular examples include:

- **Wordle**: Word guessing game
- **Tool Use Environments**: Function calling and API interaction
- **Code Execution**: Programming challenges
- **Math Problems**: Mathematical reasoning tasks
- **Multi-turn Dialogues**: Conversational AI tasks
- **Custom Environments**: Any environment you create with Verifiers

Browse the full catalog at [PrimeIntellect Environments Hub](https://docs.primeintellect.ai/tutorials-environments/environments).

## 📊 Reward System

The integration automatically handles complex reward functions:

### How It Works

1. **Automatic Extraction**: Loads rubric from the Verifiers environment
2. **Multi-Function Support**: Handles environments with multiple reward functions
3. **Weighted Composition**: Combines rewards using configurable weights
4. **Normalized Scoring**: Scales rewards to [0, 1] range

### Reward Calculation

```python
# Automatically extracted from environment
self.reward_funcs = self.rubric.get_reward_funcs()
self.reward_weights = self.rubric.get_reward_weights()

# Normalized scales
self.reward_scales = [
    weight / sum(self.reward_weights)
    for weight in self.reward_weights
]

# Calculate weighted score
weighted_rewards = [
    reward * scale
    for reward, scale in zip(rewards, self.reward_scales)
]
final_score = sum(weighted_rewards)
```

## 🔍 Parser Integration

The integration automatically uses the environment's parser:

```python
# Automatic parser extraction
self.parser = self.rubric.parser

# Parsing responses
answer_parsed = self.parser.parse_answer(completion=response_content)
```

## 📈 Evaluation

### Built-in Evaluation

```python
# Run evaluation
eval_metrics = await env.evaluate()

# Metrics returned:
# - eval/avg_total_score: Average score across all eval samples
# - Individual sample results with scores and correctness
```

### Evaluation Output

```python
{
    "score": 0.85,
    "sample": {
        "messages": [...],
        "question": "...",
        "gold_answer": "...",
        "model_parsed": "...",
        "score": 0.85,
        "correct": True,
        "finish_reason": "stop"
    }
}
```

## 🧪 Testing and Debugging

### Local Testing

```python
import asyncio
from environments.verifiers_server import VerifiersEnv

async def test():
    # Setup environment
    env_config, server_configs = VerifiersEnv.config_init()
    env_config.vf_env_name = "wordle"

    env = VerifiersEnv(config=env_config, server_configs=server_configs)
    await env.setup()

    # Test single rollout
    item = await env.get_next_item()
    result = await env.rollout_and_score_eval(
        question=item["question"],
        answer=item["answer"],
        system_prompt=env.system_prompt,
    )

    print(f"Score: {result['score']}")
    print(f"Correct: {result['sample']['correct']}")

    # Run full evaluation
    metrics = await env.evaluate()
    print(f"Eval metrics: {metrics}")

asyncio.run(test())
```

### Debug Mode

Enable detailed logging:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

## 🔧 Advanced Usage

### Custom Environment Creation

Create your own Verifiers environment and use it with Atropos:

1. **Create a Verifiers Environment**:
```python
# my_environment/environment.py
from verifiers import Environment, Rubric

class MyEnvironment(Environment):
    def __init__(self):
        super().__init__(
            name="my-env",
            system_prompt="Your system prompt here",
            rubric=MyRubric()
        )

    def get_dataset(self):
        return [...]  # Your training data

    def get_eval_dataset(self):
        return [...]  # Your eval data
```

2. **Use with Atropos**:
```python
env_config = VfEnvConfig(
    vf_env_name="my-env",
    env_args={}
)
```

### Multi-Turn Interactions

The integration supports complex multi-turn scenarios:

```python
# Environment automatically handles state and info
await self.rubric.call_reward_func(
    func=func,
    prompt=question,
    completion=messages,
    answer=answer,
    info=info,      # Environment-specific info
    state=state,    # Current state for multi-turn
)
```

### Custom Reward Weighting

Override default weights if needed:

```python
# After setup, modify weights
env.reward_weights = [0.5, 0.3, 0.2]  # Custom weights
env.reward_scales = [
    weight / sum(env.reward_weights)
    for weight in env.reward_weights
]
```

## 📝 Data Structure

### Training Data Format

```python
{
    "question": str,  # The prompt/question
    "answer": str,    # The expected answer
    "state": Any,     # Optional: Multi-turn state
    "info": Any,      # Optional: Additional metadata
}
```

### Evaluation Data Format

```python
{
    "question": str,
    "answer": str,
    "state": Any,     # Optional
    "info": Any,      # Optional
}
```

## 🚨 Common Issues & Solutions

### Environment Not Found

**Issue**: `Could not import 'wordle' environment`

**Solution**:
```bash
# Install the environment using Prime CLI
prime env install owner/environment-name
```

### Import Errors

**Issue**: `No module named 'verifiers'`

**Solution**:
```bash
# Install verifiers package
pip install "verifiers>=0.1.5.post0"
```

### API Key Issues

**Issue**: API authentication errors

**Solution**:
```bash
# Set your API key
export OPENAI_API_KEY="your-key-here"

# Or pass directly in config
APIServerConfig(api_key="your-key-here")
```

### Parser Errors

**Issue**: Parser fails to extract answer

**Solution**: Check the environment's expected format and ensure your model's responses match. Enable debug logging to see parser details.

## 🤝 Contributing

### Adding New Features

1. Fork the Atropos repository
2. Create a feature branch
3. Implement your changes to `verifiers_server.py`
4. Add tests and documentation
5. Submit a pull request

### Reporting Issues

If you encounter issues with:
- **Verifiers Environments**: Report to [PrimeIntellect/verifiers](https://github.com/PrimeIntellect-ai/verifiers)
- **Atropos Integration**: Report to [NousResearch/atropos](https://github.com/NousResearch/atropos)

## 📚 Additional Resources

- [Verifiers Documentation](https://verifiers.readthedocs.io/)
- [PrimeIntellect Environments Hub](https://docs.primeintellect.ai/tutorials-environments/environments)
- [Atropos Documentation](https://github.com/NousResearch/atropos)
- [Prime CLI Documentation](https://github.com/PrimeIntellect-ai/Prime)

## 📄 License

This environment integration is part of the Atropos training framework and follows the MIT license. See the main repository for license information.

---

**Need Help?** Join the [Nous Research Discord](https://discord.gg/nousresearch) or check the [PrimeIntellect Discord](https://discord.gg/primeintellect) for support.
