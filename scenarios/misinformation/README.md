# Misinformation Scenario

Simulates misinformation spread on a social media platform. Agents with distinct personas interact through posts, replies, likes, and boosts on a shared feed. Seed posts inject a false claim at step 0, and the simulation tracks how agents respond.

## Engine

**Social Media** (`engine: social_media`) — All agents act simultaneously each step. Each agent sees a personalized timeline (posts from users they follow), chooses one action, and all actions resolve in parallel.

## Agents

Six agents with fixed personas and goals. All use the `SocialMediaUserAgent` prefab.

| Agent | Persona | Goal |
|-------|---------|------|
| **Alice** | Sensationalist, shares shocking news without fact-checking | Get engagement with interesting/shocking content |
| **Bob** | Laid-back, posts about daily life, trusts friends | Stay connected, share positive content |
| **Charlie** | Skeptical fact-checker, questions dubious claims | Promote critical thinking, counter misinformation |
| **Diana** | Active commenter, engages with controversy, swayed by emotion | Form opinions based on what seems credible |
| **Eve** | Casual lurker, occasionally likes, watches trends | Stay informed without getting too involved |
| **Frank** | New user, trusts popular accounts and high-engagement posts | Learn the platform, find interesting content |

### Agent Components

Each `SocialMediaUserAgent` is built from these Concordia components:

1. **Memory** — Associative memory bank storing observations
2. **Instructions** — Agent identity context
3. **Persona** — Injected personality description (from config)
4. **Goal** — Injected objective (from config)
5. **Observations** — Last 5 timelines seen
6. **SituationPerception** — LLM call: "What is {name} noticing on social media?"
7. **Decision** — LLM call: "What should {name} do?" (considers all available actions)
8. **ActionFormat** — Structured action format instructions
9. **ConcatActComponent** — Concatenates all above, calls LLM to produce final action

### Available Actions

```
post      — Share a new thought
reply     — Respond to a specific post
like      — Like a post
boost     — Repost to your followers
follow    — Follow a user
unfollow  — Unfollow a user
skip      — Do nothing this turn
```

## Initial Conditions

### Follower Graph

```
Alice   → Bob, Charlie, Diana
Bob     → Alice, Charlie
Charlie → Alice, Bob, Diana
Diana   → Alice, Bob, Charlie, Eve
Eve     → Alice, Diana
Frank   → Bob, Charlie, Eve
```

### Seed Posts (step 0)

| # | Author | Content | Tags |
|---|--------|---------|------|
| 1 | Alice | "BREAKING: Scientists confirm that common household item causes serious health issues! Share before they delete this!" | misinfo_seed, health |
| 2 | Bob | "Beautiful sunset today. Sometimes it's good to just appreciate the simple things." | neutral |
| 3 | Charlie | "Just read an interesting article about critical thinking. Always verify claims before sharing!" | counter_misinfo |

## Configuration Options

In `config/scenario/misinformation.yaml`:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `social_media.timeline_limit` | 20 | Max posts shown per timeline |
| `initial_graph` | See above | Who follows whom at start |
| `seed_posts` | See above | Posts injected at step 0 |
| `agents.entities[*].params.persona` | Per agent | Personality description |
| `agents.entities[*].params.goal` | Per agent | Agent objective |

## Running

```bash
# Quick test (mock LLM)
uv run python run_experiment.py scenario=misinformation model=mock simulation.execution.max_steps=3

# With real LLM
uv run python run_experiment.py scenario=misinformation model=gpt4omini simulation.execution.max_steps=10
```
