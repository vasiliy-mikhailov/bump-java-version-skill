# bump-java-version

An [Agent Skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) that upgrades a Maven project **one Java LTS step** (8→11, 11→17, 17→21, 21→25) so it **compiles under the new JDK** and **every test that passed before still passes** — using only standard tools (JDKs, Maven, and OpenRewrite recipes from Maven Central). No project-specific scripts: the skill is a hand manual your coding agent reads and follows.

## What it does

Point a tool-using coding agent at the skill and tell it the repo and the hop. It will:

1. Record the baseline test set under the **old** JDK (the contract to conserve).
2. Make Lombok safe (≥ 1.18.30).
3. Run the official OpenRewrite *migrate-to-Java-N* recipes.
4. Apply the deterministic JDK-removal fixes as plain pom edits (re-added EE deps, surefire/Mockito floors, `--add-opens`, jacoco).
5. Compile + test under the **new** JDK and conserve every previously-passing test.
6. Do the Spring Boot 2→3 / `javax`→`jakarta` upgrade when a Java bump forces it.
7. Fall back to a troubleshooting table, or **bail honestly** with a reason.

**Done (PASS) =** `mvn compile` succeeds under the target JDK **and** no previously-passing test is lost.

## Install

### Claude Code

[Claude Code](https://github.com/anthropics/claude-code) — install it as a plugin from this repo's marketplace:

```
/plugin marketplace add vasiliy-mikhailov/bump-java-version-skill
/plugin install bump-java-version
```

It then triggers automatically when you ask Claude to upgrade or bump a Maven project's Java version.

### opencode

[opencode](https://github.com/sst/opencode) reads `AGENTS.md` and supports skills plugins such as [opencode-skillful](https://github.com/zenobi-us/opencode-skillful). Drop the skill in and reference it:

```
cp skills/bump-java-version/SKILL.md <your-project>/.bump-skill/SKILL.md
echo "Use .bump-skill/SKILL.md to bump this project's Java version." >> <your-project>/AGENTS.md
```

### OpenHands

[OpenHands](https://github.com/OpenHands/OpenHands) loads skills from `.openhands/skills/`, and from the public [extensions registry](https://github.com/OpenHands/extensions) (listing [in review](https://github.com/OpenHands/extensions/pull/311)):

```
mkdir -p <your-project>/.openhands/skills/bump-java-version
cp skills/bump-java-version/SKILL.md <your-project>/.openhands/skills/bump-java-version/
```

### Kilo Code

[Kilo Code](https://github.com/Kilo-Org/kilocode) installs skills from the [Kilo Marketplace](https://github.com/Kilo-Org/kilo-marketplace) **Skills** tab (listing [in review](https://github.com/Kilo-Org/kilo-marketplace/pull/94)), or drop it in directly:

```
cp skills/bump-java-version/SKILL.md <your-project>/.bump-skill/SKILL.md
```

### Any other agent (Cursor, Codex, Gemini CLI, …)

The skill is a single portable `SKILL.md` (markdown + YAML frontmatter, [Agent Skills spec](https://agentskills.io/)). Copy it where your agent reads skills, or reference it from your `AGENTS.md`, then prompt:

> Bump this Maven project from Java `<from>` to Java `<to>` by following `.bump-skill/SKILL.md`. Read it first, then carry out its steps yourself.

## Requirements

- The two JDKs for the hop (e.g. JDK 17 **and** 21 for a 17→21 bump), selectable via `JAVA_HOME`.
- Maven (`mvn` or the project's `./mvnw`).
- Network access to Maven Central (for the OpenRewrite recipes and any new dependencies).

## Why it's reliable

This skill wasn't hand-written. It was **evolved by an AI agent** across many iterations against a corpus of real GitHub repositories, and hardened on a **two-rung ladder**: a strong rung (Claude + Opus) first, then a production panel of **three unrelated off-the-shelf agents** — opencode, kilocode, and OpenHands — all driving the *same* model on the *identical* skill, the agent being the only variable. A change is kept only if it doesn't regress the corpus. Latest panel: **96 % test-conserving PASS** across the three agents.

## License

[MIT](LICENSE)
