# Synchronous Gym

A wrapper for OpenAI's `gym` module which enables use of multiple environments in synchrony

- [Example](#Example)
- [Resetting](#Resetting)
- [Rendering](#Rendering)
- [Cloning and Restoring](#CloningandRestoring)
- [Other](#Other)
  * [Closing](#Closing)
  * [Setting the random seed](#Settingtherandomseed)
  
<h2 id="Example">Example</h2>

To use the wrapper, simply create a gym environment as normal, than reassign the variable storing this environment to the `MultiGymWrapper` object as follows:
```python
import gym
from wrapper import MultiGymWrapper

# Make a standard gym environment
env = gym.make("Qbert-v0")

# Wrap the environment inside the multi-agent wrapper object
# The parameter n specifies the number of simultaneous simulations
env = MultiGymWrapper(env, n = 8)

# Run a random episode on all 8 environments simultaneously
states = env.reset()

while True:
    actions = env.action_space.sample()
    states, rewards, terminals, infos = env.step(actions)
    env.render()
    if any(terminals):
        break
    
# Close all 8 open simulations
env.close()
```

When writing your implementation with the wrapper, keep in mind that the standard functions from the `gym` module will instead return a list in most cases, one result for each environment. For example, when sampling actions or taking steps, the results are lists. Thankfully, the wrapper overrides some of the components of the `gym` module, enabling methods such as `Env.step()` to take a list of actions as input.

<h2 id="Resetting">Resetting</h2>

This is self-explanatory. Simply calling `.reset()` on your `MultiGymWrapper` object will automatically iterate through each simulation, calling `Env.reset()` on each of them. While the implementation of this method in the `gym` module returns an initial RGB array, this returns the initial RGB array for each open simulation:
```python
env = gym.make("Qbert-v0")
env = MultiGymWrapper(env, n = 8)
states = env.reset()
print (len(states), states[0].shape)
>> 8 (210, 160, 3)
```

<h2 id="Rendering">Rendering</h2>

With the wrapper, you can choose to either render just **one** environment, or all of them.

You can perform either by simply specifying the approperiate parameter for the argument `which` of the `render` method:
- `which = 'one'`: Renders solely the first of the set of open simulations (default)
- `which = 'all'`: Renders all of the open simulations; this can get a bit messy

Here's an example, assuming `env` is a `MultiGymWrapper` object:
```python
# Renders solely the first of the set of open simulations
env.render(mode = 'human')

# Render all of the open simulations
env.render(mode = 'human', which = 'all')

# Rather than displaying the rendering of the first simulation,
# this will return an RGB array of the frame which would otherwise
# be displayed
rgb_frame = env.render(mode = 'rgb_array')

# This will return a list of RGB arrays, one for the current frame of each simulation
rgb_frames = env.render(mode = 'rgb_array')
```

Here's a code sample:
```python
env = gym.make("Qbert-v0")
env = MultiGymWrapper(env, n = 8)
states = env.reset()
env.render() # default is which='one'
```
<a href="https://imgbb.com/"><img src="https://i.ibb.co/mTdh37s/window-qbert.png" alt="window-qbert" border="0"></a>

Of course, the above window would actually close immediately if you run the above code sample because the script ended immediately afterwards with no frame to replace it.

<h2 id="CloningandRestoring">Cloning and Restoring</h2>

All this is very intuitive; most methods simply return a list of the results you would expect, one for each simulation. \
This, too, works in the way you'd expect:

- `clone_full_states` and `clone_states` returns a list of cloned states for each environment as follows:
  ```python
  def clone_full_states(self):
      return [env.env.clone_full_state() for env in self.envs]
      
  def clone_states(self):
      return [env.env.clone_state() for env in self.envs]
  ```
  
- `restore_full_states` and `restore_states` work identically. They take a list of states as input.
  ```python
  def restore_full_states(self, states):
      for state, env in zip(states, self.envs):
          env.env.restore_full_state(state)

  def restore_states(self, states):
      for state, env in zip(states, self.envs):
          env.env.restore_state(state)
  ```
  
Note that the only difference between the wrapper class and the standard `gym` module is how you call these functions. In `gym`:
- `Env.env.clone_full_state()`
- `Env.env.clone_state()`
- `Env.env.restore_full_state()`
- `Env.env.restore_state()`

In the wrapper, you simply call the function on the object (also, the function name is plural):
- `Env.clone_full_states()`
- `Env.clone_states()`
- `Env.restore_full_states()`
- `Env.restore_states()`

Here's a code sample:
```python
env = gym.make("Qbert-v0")
env = MultiGymWrapper(env, n = 8)
env.reset()
rams = env.clone_states()
print (rams)
>> [array([247,  60,   6, ...,   0,   0,   0], dtype=uint8), array([247,  60,   6, ...,   0,   0,   0], dtype=uint8), array([247,  60,   6, ...,   0,   0,   0], dtype=uint8), array([247,  60,   6, ...,   0,   0,   0], dtype=uint8), array([247,  60,   6, ...,   0,   0,   0], dtype=uint8), array([247,  60,   6, ...,   0,   0,   0], dtype=uint8), array([247,  60,   6, ...,   0,   0,   0], dtype=uint8), array([247,  60,   6, ...,   0,   0,   0], dtype=uint8)]
env.restore_states(rams)
```

<h2 id="Other">Other</h2>

Essentially all other functions work as you'd expect. Here are some other examples:

<h3 id="Closing">Closing</h3>

To close all of the simulations, simply call `.close()` on the `MultiGymWrapper` object:
```python
env.close()
```
<h3 id="Settingtherandomseed">Setting the random seed</h3>

In the standard `gym`, you specify a non-negative integer as input. In the `MultiGymWrapper`, you also just specify one non-negative integer. It will apply to all of the open simulations.
```python
seed = 42
env.seed(seed)
```
