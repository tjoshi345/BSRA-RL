# CartPole report

Replace each **TODO** with your own observations. Keep this report short; link code,
compact results, and images instead of pasting logs.
For each stage, explain which tool or function uses your inputs, where the
results come from, and what uses them next. Point to the relevant code or
documentation; a few sentences or an annotated arrow diagram is enough.

To link a file in Markdown, use `[link text](relative/path/to/file)`.
For example, `[Environment code](../onboarding/env.py)` becomes
[Environment code](../onboarding/env.py). Links are relative to this report's
`docs/` folder; use `../` to reach the repository root.
To display an image, add `!`: `![Model screenshot](../results/model.png)`.

## Setup

- Member: **Twisha**
- OS: **MacOS**
- Setup diagnostic, viewer, and RGB results; any fix needed: **Needed to change a few lines at the end of viewer.py after finally: to skip any threads that couldn't be recognized**
- Fork URL and working branch: **Not too many issues**
- Before starting, read the [toolchain overview](../resources/toolchain.md).
  What role does each of MuJoCo, Gymnasium, Stable-Baselines3, and TensorBoard
  play in this exercise? Describe how they work together in your own words: **MuJoCo simulates the environment for the agent and can provide a visual for the viewer; Gymnasium defines the abilities, rewards, and overall setup for the agent to train with; SB3 handles all of the loops and how the agent trains and improves using the rewards; and TensorBoard shows you all of the training data**

## Model and task

### Model (Stage 2)
git diff origin/main origin/onboarding
- Which tool loads `scene.xml` and its included `cartpole.xml`, and what does it
  create from them? Point to the loading call in `scripts/view_model.py`. Which
  tool computes the motion you see in the viewer when a control is applied? **Mujoco is used to load scene and it creates a model using them. 'model = mujoco.MjModel.from_xml_path(str(args.xml.resolve()))'**
- Which XML file owns the mechanism, and how does the include connect it to the
  scene? Explain the slide/hinge axes, unactuated pole, box half-extents, and
  degrees versus radians. Link a small model screenshot (`../results/model.png`): **cartpole has all of the mechanisms in it, and it is connected to the scene xml through the include line. axis describes the motion of the item, x y z, so the slider moves in the x direction and the hinge moves in the y direction. the unacuated pole means that it doesnt have anything acting on it to move, so it'll only move if something hits it. the box size is half the length so you need to divide the values by half. you have to make sure you set it to degress because mujoco defaults to radians.**

### Environment (Stage 3)

Answer each row in a few sentences, using the source linked in
[Stage 3](ONBOARDING.md#3-environment-setup-and-inspection). Name the function or
setting that supports your answer, and distinguish MuJoCo's role from Gymnasium's.

| Topic        | Question                                                                                                                                                | Your answer |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| Observations | What are the four values returned to the policy, in order and with units? Where do those values come from, and how does `_get_obs()` assemble them?     | **it takes the cart position (m), pole position (radians), cart velocity (m/s), and pole velocity (rad/s). it comes from the array returned in the _get_obs method**    |
| Actions      | What does the action control, what is its allowed range, and how does it reach the motor?                                                               | **action controls the force applied to the cart and its range is -3 to 3. it reaches the motor through ctrl/data.ctrl which is connectred to the actuator**    |
| Physics      | Which tool computes motion from your XML model, and how does Gymnasium ask it to advance? How much simulated time passes per action?                    | **Mujoco computes the motion and it uses step() to advance. 0.04 seconds pass, it takes 0.02 from the scene xml and multiplies it by 2**    |
| Reset        | What does `reset_model()` change at the start of an episode? How are the starting values randomized, and what does using the same reset seed reproduce? | **it resets the setup and then randomizes by how much it'll move the pole to put the setup into motion. in the reset_model method it adds a random number to the positions and velocities using np_random.uniform. using the same reset seed starts it the same every time**    |
| Reward       | What is the exact reward rule, which code computes it, and does it use the state before or after the action?                                            | **+1 is given for reach timestep that the pole is upright and it is comptated in step(). it uses the state after**    |

- Environment test and random-rollout results; action-range warning: **passed the 11 tests and got one warning**

## Training

- Run name / source commit / training seed: **first-run / 81d2daf4e422b161ab83c9996c237e0b6c4a0875 / 0**
- Requested and actual steps / elapsed time / device: **100000 and 100352 / 27.78760958299972 / cpu**
- Configuration and versions: **/BSRA-RL/results/first-run/run.json** link `../results/<run>/run.json`
- Learning curve: **/BSRA-RL/results/first-run/learning-curve.png** embed `../results/<run>/learning-curve.png`

- What do SB3, PyTorch, and `Monitor` each do here? How does SB3 use the environment
  and settings you pass to PPO? **sb3 helps create the ppo, pytorch has the logic to build the ppo, and monitor summarizes the results**
- What do the curve's axes, averaging, and smoothing mean, and what evidence of
  improvement, variability, or a plateau do you see? **the x axis is timesteps, the y axis is average reward per episode, the smoothing affects the graph. the graph became steeper and more consistent.**

## Evaluation

Use reset seeds 10000–10019, deterministic PPO actions, and random action seeds
`reset seed + 20000`. Standard deviations describe episodes (`ddof=0`).

| Agent | Episodes | Return mean ± std | Length mean ± std | Time-limit fraction |
| --- | --- | --- | --- | --- |
| Random | 20 | 4.8 ± 2.6570660511172846 | 5.8 ± 2.6570660511172846 | 0.0 |
| PPO | 20 | 1000.0 ± 0.0 | 1000.0 ± 0.0 | 1.0 |

- Per-episode CSV and summary JSON: **csv: /BSRA-RL/results/first-run/episodes.csv json: /BSRA-RL/results/first-run/summary.json** links
- Predetermined PPO rollout (seed 10000): **the pole wobbled a little but then didn't move** briefly describe the behavior you observed
- Limitation or failure, and a specific diagnosis if learning was weak: **I think the learning was fine**

- Why does this evaluation differ from a training curve, and what can one
  trained seed not establish? What carries over to humanoid soccer, and what is
  missing (for example, contacts, partial observations, or sim-to-real transfer)? **one trained seed cannot account for random exceptions so you would need for it to go through a lot more training and random events so that it can learn and gain more experience**

## Review

**TODO:** Record exact setup, test, train, and evaluate commands, plus any rendering
environment variable.

Keep the saved policy, TensorBoard logs, and rollout video locally and share them
directly with the RL lead if requested. No file uploads, download links, or GitHub
release are required for these files.

- Automated test results: **TODO**
- Manual model/render/video checks: **TODO**
- Fork PR link and review notes (record merge in the PR): **TODO**

## Feedback

- Did you learn anything from this onboarding? What was new, or what became
  clearer? If little was new, say so: **TODO**
- What improvements would make the onboarding easier to follow or more useful?
  Mention any confusing instructions, missing background, or unnecessary work: **TODO**
