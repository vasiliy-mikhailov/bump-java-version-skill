# bump-java-version

[Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) that upgrade a Maven **or Gradle** project **one Java LTS step** (8→11, 11→17, 17→21, 21→25) so it **compiles under the new JDK**, **every test that passed before still passes**, and the **effective compiler target is raised to the new LTS**: using only standard tools (JDKs, Maven or Gradle, and OpenRewrite recipes from Maven Central). No project-specific scripts: each skill is a hand manual your coding agent reads and follows.

This release ships **one skill per LTS hop, plus a router**:

- **`detect-java-version`**: reads the project's *real* current Java level (its declared bytecode target across all modules, not just the build toolchain) and dispatches to the right hop, or reports that the project should **not** be bumped (a deliberately low-target library/plugin).
- **`bump-java-8-to-11`**, **`bump-java-11-to-17`**, **`bump-java-17-to-21`**, **`bump-java-21-to-25`**: each performs one fixed LTS hop.

## What it does

Point a tool-using coding agent at the skills and tell it the repo. `detect-java-version` reads the real bytecode target and routes to the matching hop skill, which:

1. Records the baseline test set under the **old** JDK (the contract to conserve).
2. Applies **proactive** safety steps up front, gated on what the project uses, e.g. floor Lombok to the JDK-safe version **and** force annotation processing; bump Kotlin (and version-locked KSP) to a release that emits the new bytecode.
3. Runs the official OpenRewrite *migrate-to-Java-N* recipes, sets the compiler target, and bumps the Gradle wrapper when the new JDK requires it.
4. Compiles + tests under the **new** JDK, conserving every previously-passing test and confirming the effective compiler target is the new LTS (across every module).
5. Works a per-hop troubleshooting list (ByteBuddy/Mockito, JaCoCo, Kotlin/KSP, multi-module target propagation, `javax`→`jakarta` / Spring Boot, …), or **bails honestly** with a labelled reason.

**Done (PASS) =** compiles under the target JDK, no previously-passing test lost, and the effective compiler target is the new LTS.

## Install

### Claude Code

[Claude Code](https://github.com/anthropics/claude-code), install as a plugin from this repo's marketplace (all five skills come with it):

```
/plugin marketplace add vasiliy-mikhailov/bump-java-version-skill
/plugin install bump-java-version
```

They trigger automatically when you ask Claude to detect or bump a Maven or Gradle project's Java version.

### opencode / OpenHands / Kilo Code / any other agent

Each skill is a portable `SKILL.md` (markdown + YAML frontmatter, [Agent Skills spec](https://agentskills.io/)). Copy the **whole set** where your agent reads skills, so the router can dispatch to the right hop:

```
# opencode / generic: copy all skills, then reference them from AGENTS.md
cp -r skills/* <your-project>/.bump-skills/
echo "Use .bump-skills/ to detect and bump this project's Java version." >> <your-project>/AGENTS.md

# OpenHands (reads .openhands/skills/, and the public extensions registry)
cp -r skills/* <your-project>/.openhands/skills/

# Kilo Code: install from the Kilo Marketplace Skills tab, or drop the set in directly
cp -r skills/* <your-project>/.bump-skills/
```

Then prompt:

> Detect this Maven or Gradle project's Java version and bump it one LTS step, following the `bump-java-*` skills. Read the matching skill first, then carry out its steps yourself.

## Requirements

- The two JDKs for the hop (e.g. JDK 17 **and** 21 for a 17→21 bump), selectable via `JAVA_HOME` (or Gradle's `-Dorg.gradle.java.home`).
- Maven (`mvn` or the project's `./mvnw`) **or** Gradle (the project's `./gradlew`), each skill auto-detects which (`pom.xml` → Maven, else `build.gradle`/`.kts` → Gradle).
- Network access to Maven Central (for the OpenRewrite recipes and any new dependencies).

## Why it's reliable

These skills weren't hand-written. They were **evolved by an AI agent** across many iterations against a corpus of real GitHub repositories, and hardened on a **two-rung ladder**: a strong rung (Claude + Opus) first, then a production panel of **three unrelated off-the-shelf agents**: opencode, kilocode, and OpenHands, all driving the *same* model on the *identical* skill, the agent being the only variable. A change is kept only if it doesn't regress the corpus.

## License

[MIT](LICENSE)
