# Synchronus Gym
A wrapper for OpenAI gym which enables use of multiple environments in synchrony

# Example
To use the wrapper, simply create a gym environment as normal, than reassign the variable storing this environment to the MultiGymWrapper object as follows:
```
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

# Rendering
With the wrapper, you can choose to either render just **one** environment, or all of them.

You can perform either by simply specifying the approperiate parameter for the argument `which` of the `render` method:
- `which = 'one'`: Renders solely the first of the set of open simulations (default)
- `which = 'all'`: Renders all of the open simulations; this can get a bit messy

Here's an example, assuming `env` is a `MultiGymWrapper` object:
```
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

# Cloning and Restoring
All this is very intuitive; most methods simply return a list of the results you would expect, one for each simulation. \
This, too, works in the way you'd expect /

- `clone_full_states` and `clone_states` returns a list of cloned states for each environment as follows:
  ```
  def clone_full_states(self):
      return [env.env.clone_full_state() for env in self.envs]
      
  def clone_states(self):
      return [env.env.clone_state() for env in self.envs]
  ```
  
- `restore_full_states` and `restore_states` work identically. They take a list of states as input.
  ```
  def restore_full_states(self, states):
      for state, env in zip(states, self.envs):
          env.env.restore_full_state(state)

  def restore_states(self, states):
      for state, env in zip(states, self.envs):
          env.env.restore_state(state)
  ```
