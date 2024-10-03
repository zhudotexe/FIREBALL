# FIREBALL: A Dataset of Dungeons and Dragons Actual-Play with Structured Game State Information

> Dungeons & Dragons (D&D) is a tabletop roleplaying game with complex natural language interactions between players and
> hidden state information. Recent work has shown that large language models (LLMs) that have access to state
> information can generate higher quality game turns than LLMs that use dialog history alone. However, previous work
> used game state information that was heuristically created and was not a true gold standard game state. We present
> FIREBALL, a large dataset containing nearly 25,000 unique sessions from real D&D gameplay on Discord with true game
> state info. We recorded game play sessions of players who used the Avrae bot, which was developed to aid people in
> playing D&D online, capturing language, game commands and underlying game state information. We demonstrate that
> FIREBALL can improve natural language generation (NLG) by using Avrae state information, improving both automated
> metrics and human judgments of quality. Additionally, we show that LLMs can generate executable Avrae commands,
> particularly after finetuning.



[Read the paper here!](https://aclanthology.org/2023.acl-long.229/)

[Download the dataset!](https://datasets.mechanus.zhu.codes/fireball-anonymized-nov-28-2022-kfdjl.tar.gz) (7.7GB; last
updated Nov 28, 2022)
**Note**: This dataset is now on HuggingFace: [https://huggingface.co/datasets/lara-martin/FIREBALL](https://huggingface.co/datasets/lara-martin/FIREBALL)

[Dataset preprocessing code](https://github.com/zhudotexe/FIREBALL-data-processing)

[Avrae source code](https://github.com/avrae/avrae)

## Citation

```bibtex
@inproceedings{FIREBALL,
    title = "FIREBALL: A Dataset of \textit{Dungeons and Dragons} Actual-Play with Structured Game State Information",
    author = "Zhu, Andrew and Aggarwal, Karmanya and Feng, Alexander and Martin, Lara J. and Callison-Burch, Chris",
    booktitle = "Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (ACL)",
    month = 07,
    year = "2023",
    url = "https://aclanthology.org/2023.acl-long.229/",
    doi = "10.18653/v1/2023.acl-long.229",
    eprint = "2305.01528",
    archivePrefix = "arXiv",
    address = "Toronto, Canada"
}
```

## This Repository

FIREBALL is pretty large, so this repo contains only a single combat instance in the same format found in the full
release of FIREBALL. **To download the full FIREBALL dataset, click the link above.**

`raw.jsonl` consists of all the events in the combat instance, with one event per line.

`filtered_triples.jsonl` consists of the results of running the raw events through the distillation process specified in
the paper, with each line containing a (before, commands, after) triple along with associated metadata such as the
combat states.

All user IDs and usernames have been randomized (by way of a hash function) to preserve anonymity.

FIREBALL is released for research purposes only under a CC-BY-4.0 license.

## Event Schema (raw.jsonl)

Each line contains a recorded event, which must be one of the following:

```text
- message
- alias_resolution
- snippet_resolution
- command
- button_press
- automation_run
- combat_state_update
```

The meanings of these events are discussed in Appendix B of the FIREBALL paper. Each event contains at least the
following keys:

- `combat_id`: Used for data partitioning, should be the same across all events in a gameplay instance.
- `event_type`: One of the above. Defines the other keys available in this event body.
- `timestamp`: The time this event was generated, as a UNIX timestamp.

The remaining event schema is defined [here](https://github.com/avrae/avrae/blob/v4.2.2/cogs5e/initiative/upenn_nlp.py).

## Filtered Triples Schema (filtered_triples.jsonl)

Each line contains a filtered triple, each of which includes the following keys:

```text
{
    "speaker_id": The anonymized user ID of the user who sent the commands in the triple.
    "before_utterances": A list of strings corresponding to the "preceding" utterances in the triple.
    "combat_state_before": A list of normalized actor states (see below) for each actor in the combat instance at the instant before the command was run.
    "current_actor": (nullable) The normalized actor state of the actor whose turn it currently is.
    "commands_norm": A list of strings corresponding to the "commands" portion of the triple.
    "automation_results": A mechanically generated list of strings representing the results of running the action in the Avrae engine.
    "caster_after": The normalized actor state of the actor who ran the action(s), which may or may not be the current actor.
    "targets_after": A list of normalized actor states for each actor who was targeted by the action.
    "combat_state_after": A list of normalized actor states for each actor in the combat instance at the instant after the command was run.
    "after_utterances": A list of strings corresponding to the "folllowing" utterances in the triple.
    "utterance_history": The last 5 messages in the chat history before the command was run.
    "before_idxs": A list of integers corresponding to the index of the "message" events containing the "preceding" utterances in the raw event file.
    "before_state_idx": The index of the "combat_state_update" event in the raw event file that was used to derive "combat_state_before".
    "command_idxs": The indexes of the "command" events corresponding to the "commands_norm" key.
    "after_state_idx": The index of the "combat_state_update" event corresponding to the "combat_state_after" key.
    "after_idxs": The indexes of the "message" events corresponding to the "after_utterances" key.
    "embed_idxs": (nullable, same length as "automation_results") The indexes of "message" events corresponding to rich results shown to players on Discord for each result in the "automation_results" key.
}
```

### Normalized Actor State

The normalized actor state is only a subset of the available actor information, corresponding to the information we
used for our engineering experiments for the FIREBALL paper. For a full list of available actor information, see table 6
in the FIREBALL paper.

```text
{
    "name": The name of the actor.
    "hp": The numerical and narrative hit points (e.g. "<12/34; Bloodied>").
    "class": The actor's class(es) and level(s), if applicable (e.g. "Fighter 3")
    "race": The actor's race, if applicable (e.g. "Mountain Dwarf", "Adult Red Dragon").
    "attacks": A list of the actor's available attack names.
    "spells": A list of the actor's available spells.
    "actions": A list of the actor's available special abilities.
    "effects": A list of any temporary effects on the actor (e.g. "Stunned").
    "description": The actor's narrative description (if available).
    "controller_id": The anonymized user ID of this actor's controller.
}
```
