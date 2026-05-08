# aicrate - containerize your AI agents

`aicrate` is a Python CLI tool that packages and orchestrates AI agents, skills, and MCP (Model Context Protocol) servers as OCI artifacts in isolated container environments. It enables you to build reproducible, shareable AI workspaces by bundling these artifacts and tools into unified pods.

## Prerequisites

- python 3.9+
- [podman](https://podman.io/)
- [git](https://git-scm.com/)

## Installation

Simply install `aicrate` via pip:

```bash
$ pip install aicrate
```

## Features

`aicrate` provides a [podman](https://podman.io/)-like CLI to [build](#build---put-ai-agents-and-skills-into-oci-artifacts), list and [push and pull](#push-and-pull---using-oci-registries-for-ai-agents-and-skills) OCI artifacts of AI agents and skills as well as [running](#run---running-ai-agents-and-a-workbox) them in a unified pod.

### build - Put AI agents and skills into OCI artifacts

The `aicrate build` command enables packing AI agents and skills into OCI artifacts. It also provides a simple way to build a workbox container image with claude code installed.

Assume the following folder structure:

```
obra-superpowers
├── agents/
│   └── code-reviewer.md
├── skills/
│   ├── brainstorming/
│   │   ├── scripts/
│   │   ├── SKILL.md
│   │   ├── spec-document-reviewer-prompt.md
│   │   └── visual-companion.md
├── ...
```

The agent and skill can be wrapped into an OCI artifact like this:

```bash
# Build the OCI artifact for the brainstorming skill
$ aicrate build skill --dir obra-superpowers/skills/brainstorming

# Build the OCI artifact for the code-reviewer agent
$ aicrate build agent --dir obra-superpowers/agents

# List the OCI artifacts built just now
$ aicrate ls
┌────────┬─────────────────────────────────────────────┬─────────────────────────────────────┬──────────────────────────────────────────┐
│ Type   │ Name                                        │ Remote                              │ Version                                  │
├────────┼─────────────────────────────────────────────┼─────────────────────────────────────┼──────────────────────────────────────────┤
│ Skill  │ quay.io/aicrate/brainstorming:latest        │ git@github.com:obra/superpowers.git │ eafe962b18f6c5dc70fb7c8cc7e83e61f4cdde06 │
│ Agent  │ quay.io/aicrate/agents:latest               │ git@github.com:obra/superpowers.git │ eafe962b18f6c5dc70fb7c8cc7e83e61f4cdde06 │
└────────┴─────────────────────────────────────────────┴─────────────────────────────────────┴──────────────────────────────────────────┘
```

### push and pull - Using OCI registries for AI agents and skills

Using the `aicrate pull` AI agents and skills wrapped into OCI artifacts can be pulled from an OCI registry. Complementary, the `aicrate push` enables sending them to the registry for later pulling.

```bash
# Push a built OCI artifact to registry
$ aicrate push artifact quay.io/aicrate/brainstorming:latest

# Pull a built OCI artifact from registry
$ aicrate pull artifact quay.io/aicrate/brainstorming:latest
```

### run - Running AI agents and a workbox

AI agents and skills wrapped as OCI artifacts can be used by `aicrate run` to extend the containerized AI with selected tools for minimal, efficient intial context. In the simplest case, only the containerized AI is started as a "workbox":


```bash
# Run a containerized AI agent with the current directory mounted into 
# the container as workspace and no additional skills or agents
$ aicrate run
bash-5.3 # 
```

In order to extend the workbox with agents and skills, create a [configuration file](#default-configuration). For example, [examples/aicrate.simple.yml](./examples/aicrate.simple.yml) adds skills and an agent to the workbox. Use it like this:

```bash
$ aicrate run --config ./examples/aicrate.simple.yml
bash-5.3 # 
```

**Note:** After running `aicrate run`, the user is automatically dropped into the mounted directory of the workbox - container internal the `/workspace`. The detach option `-d` runs it without dropping the user directly into the container.


## Configuration

For setting up a workbox with defined AI agents, skills and (local) MCP servers, `aicrate` uses a yaml configuration. 

For example, using the following configuration file `aicrate` will 
- start a local pod with a workbox container (incl. claude)
- mount the local home directory into the workbox (in container path `/workspace`)
- and provide the listed three AI skills and the code-reviewer agent

```yaml
# aicrate.yml
workbox:
  image: quay.io/aicrate/claudebox:latest
  workspace: ~/
  skills:
    - quay.io/aicrate/anthropic/claude-api:latest
    - quay.io/aicrate/anthropic/pdf:latest
    - quay.io/aicrate/obra/test-driven-development:latest
  agents:
    - quay.io/aicrate/agency/engineering-code-reviewer:latest
```

See [aicrate on quay.io](https://quay.io/organization/aicrate) for pre-built and ready to pull artifacts.


### Default configuration

By using [aicrate run](#run---running-ai-agents-and-a-workbox), the default configuration file will always be used. It is located in:

```
~/.local/share/aicrate/aicrate.yml
```

and **needs to be created manually**. If existing, `aicrate` will merge the content of the default configuration file with the configuration specified by the CLI. So the order of precedence for configuration is as follows:

```
default config ─> config from CLI ─> CLI parameter
```


## Similar projects

- [LobsterTrap/puzzlepod](https://github.com/LobsterTrap/puzzlepod) \
`PuzzlePod` and aicrate leverage Podman and systemd and run the agents inside a container. However, while PuzzlePod adds a governance layer on top (e.g. automated commit/rollback decisions), `aicrate` leverages [OCI artifacts](https://edu.chainguard.dev/open-source/oci/what-are-oci-artifacts/) to manage skills and agents in OCI registries.
- [NVIDIA/OpenShell](https://github.com/NVIDIA/OpenShell) \
`OpenShell` provides a safe, private runtime for autonomous AI agents. Its goal are multi-tenant enterprise deployments while `aicrate` is intended for local usage by a single developer.
- [fletchgpc/agentbox](https://github.com/fletchgqc/agentbox) \
`agentbox` provides a container-based environment for automated AI agents for more safety. `aicrate` adds the concepts of skills and agents as OCI artifacts to use, share and maintain them.
