# Gymnasium Migration Summary

This document summarizes the migration from OpenAI Gym (v0.21) to Gymnasium (v0.26+) in the LIBERO codebase.

## Overview

The migration was completed to ensure forward compatibility with modern RL libraries and to use the actively maintained Gymnasium package instead of the deprecated OpenAI Gym.

## Changes Made

### 1. Dependencies (`requirements.txt`)
- **Before**: `gym==0.25.2`
- **After**: `gymnasium>=0.28.1`

### 2. Import Statements (`libero/libero/envs/venv.py`)
- **Before**: `import gym`
- **After**: `import gymnasium as gym`

### 3. Type Annotations (`libero/libero/envs/venv.py`)
- Removed old type definitions: `gym_old_venv_step_type`, `gym_new_venv_step_type`
- Added new unified type: `gym_venv_step_type = Tuple[np.ndarray, np.ndarray, np.ndarray, np.ndarray, np.ndarray]`
- This represents: `(observations, rewards, terminated, truncated, info)`

### 4. Environment Reset (`libero/libero/envs/env_wrapper.py`)
- Updated `reset()` method to accept optional `seed` parameter
- **Before**: `def reset(self):`
- **After**: `def reset(self, seed=None, **kwargs):`
- Seeding now handled via: `env.reset(seed=seed)` instead of `env.seed(seed)`
- Legacy `seed()` method kept for compatibility with robosuite

### 5. Step Function Returns
Updated all `env.step()` calls to handle 5 return values instead of 4:

#### Files Updated:
- `libero/lifelong/metric.py`
- `libero/lifelong/evaluate.py`
- `scripts/create_dataset.py`
- `benchmark_scripts/render_single_task.py`

#### Changes:
- **Before**: `obs, reward, done, info = env.step(action)`
- **After**: `obs, reward, terminated, truncated, info = env.step(action)`
- Added: `done = terminated | truncated` where the `done` flag is needed

### 6. Episode Termination Logic
- `terminated`: True when episode ends naturally (e.g., task completed or failed)
- `truncated`: True when episode ends due to time limit or other artificial constraints
- `done = terminated | truncated` for backward compatibility with existing logic

### 7. Documentation Updates
- Updated `README.md` to show correct Gymnasium API usage
- Updated example code snippets to use new step/reset signatures

### 8. Notebooks
Updated Jupyter notebooks with the new API:
- `notebooks/quick_walkthrough.ipynb`
- `notebooks/quick_guide_algo.ipynb`
- `notebooks/custom_object_example.ipynb` (no changes needed)

## API Reference

### Old Gym API (v0.21)
```python
import gym

env = gym.make("env-id")
env.seed(42)
obs = env.reset()

for step in range(max_steps):
    action = policy(obs)
    obs, reward, done, info = env.step(action)
    if done:
        break
```

### New Gymnasium API (v0.26+)
```python
import gymnasium as gym

env = gym.make("env-id")
obs, info = env.reset(seed=42)

for step in range(max_steps):
    action = policy(obs)
    obs, reward, terminated, truncated, info = env.step(action)
    if terminated or truncated:
        break
```

## Compatibility Notes

1. **robosuite Environments**: The underlying robosuite environments still use the old Gym API internally. Our wrapper layer (`env_wrapper.py`) handles the translation, accepting the new Gymnasium API externally while maintaining compatibility with robosuite.

2. **Seed Behavior**: The `seed()` method is retained in `ControlEnv` for robosuite compatibility, but the preferred method is now `reset(seed=seed)`.

3. **VectorEnv**: The vectorized environment (`venv.py`) now returns 5-tuples from `step()` consistently across all environments.

## Testing Recommendations

After this migration, the following should be tested:

1. **Basic Environment Usage**: Verify environments can be created, reset, and stepped through
2. **Seeding**: Ensure deterministic behavior when using `reset(seed=N)`
3. **Training Loops**: Verify lifelong learning algorithms work with the new API
4. **Evaluation**: Ensure evaluation scripts produce correct metrics
5. **Data Collection**: Verify demonstration collection scripts still work

## Files Modified

### Core Libraries
- `requirements.txt`
- `libero/libero/envs/venv.py`
- `libero/libero/envs/env_wrapper.py`

### Scripts
- `libero/lifelong/evaluate.py`
- `libero/lifelong/metric.py`
- `scripts/create_dataset.py`
- `benchmark_scripts/render_single_task.py`

### Documentation
- `README.md`
- `notebooks/quick_walkthrough.ipynb`
- `notebooks/quick_guide_algo.ipynb`

## Migration Checklist

- [x] Updated dependencies
- [x] Updated imports
- [x] Updated type hints
- [x] Updated reset() signatures
- [x] Updated step() return value handling
- [x] Updated seeding mechanism
- [x] Updated documentation
- [x] Updated notebooks
- [x] All Python files pass syntax check

## Known Limitations

1. **Robosuite Dependency**: The migration assumes robosuite continues to work with its current API. If robosuite migrates to Gymnasium, the wrapper layer may need adjustments.

2. **Interactive Scripts**: Scripts like `collect_demonstration.py` that don't capture step/reset return values were left unchanged as they work correctly with the pass-through wrapper.

## Future Work

1. Monitor robosuite for potential Gymnasium migration
2. Consider adding automated tests for the Gymnasium API
3. Update any CI/CD pipelines to use gymnasium
4. Consider contributing Gymnasium compatibility to robosuite upstream

## References

- [Gymnasium Documentation](https://gymnasium.farama.org/)
- [Gymnasium Migration Guide](https://gymnasium.farama.org/content/migration-guide/)
- [LIBERO Project](https://libero-project.github.io/)
