---
title: InMemoryLookupKB
teaser:
  The default implementation of the KnowledgeBase interface. Stores all
  information in-memory.
tag: class
source: spacy/kb/kb_in_memory.pyx
new: 3.5
---

The `InMemoryLookupKB` class inherits from [`KnowledgeBase`](/api/kb) and
implements all of its methods. It stores all KB data in-memory and generates
[`Candidate`](/api/kb#candidate) objects by exactly matching mentions with
entity names. It's highly optimized for both a low memory footprint and speed of
retrieval.

## InMemoryLookupKB.\_\_init\_\_ {#init tag="method"}

Create the knowledge base.

> #### Example
>
> ```python
> from spacy.kb import InMemoryLookupKB
> vocab = nlp.vocab
> kb = InMemoryLookupKB(vocab=vocab, entity_vector_length=64)
> ```

| Name                   | Description                                      |
| ---------------------- | ------------------------------------------------ |
| `vocab`                | The shared vocabulary. ~~Vocab~~                 |
| `entity_vector_length` | Length of the fixed-size entity vectors. ~~int~~ |

## InMemoryLookupKB.entity_vector_length {#entity_vector_length tag="property"}

The length of the fixed-size entity vectors in the knowledge base.

| Name        | Description                                      |
| ----------- | ------------------------------------------------ |
| **RETURNS** | Length of the fixed-size entity vectors. ~~int~~ |

## InMemoryLookupKB.add_entity {#add_entity tag="method"}

Add an entity to the knowledge base, specifying its corpus frequency and entity
vector, which should be of length
[`entity_vector_length`](/api/kb_in_memory#entity_vector_length).

> #### Example
>
> ```python
> kb.add_entity(entity="Q42", freq=32, entity_vector=vector1)
> kb.add_entity(entity="Q463035", freq=111, entity_vector=vector2)
> ```

| Name            | Description                                                |
| --------------- | ---------------------------------------------------------- |
| `entity`        | The unique entity identifier. ~~str~~                      |
| `freq`          | The frequency of the entity in a typical corpus. ~~float~~ |
| `entity_vector` | The pretrained vector of the entity. ~~numpy.ndarray~~     |

## InMemoryLookupKB.set_entities {#set_entities tag="method"}

Define the full list of entities in the knowledge base, specifying the corpus
frequency and entity vector for each entity.

> #### Example
>
> ```python
> kb.set_entities(entity_list=["Q42", "Q463035"], freq_list=[32, 111], vector_list=[vector1, vector2])
> ```

| Name          | Description                                                      |
| ------------- | ---------------------------------------------------------------- |
| `entity_list` | List of unique entity identifiers. ~~Iterable[Union[str, int]]~~ |
| `freq_list`   | List of entity frequencies. ~~Iterable[int]~~                    |
| `vector_list` | List of entity vectors. ~~Iterable[numpy.ndarray]~~              |

## InMemoryLookupKB.add_alias {#add_alias tag="method"}

Add an alias or mention to the knowledge base, specifying its potential KB
identifiers and their prior probabilities. The entity identifiers should refer
to entities previously added with [`add_entity`](/api/kb_in_memory#add_entity)
or [`set_entities`](/api/kb_in_memory#set_entities). The sum of the prior
probabilities should not exceed 1. Note that an empty string can not be used as
alias.

> #### Example
>
> ```python
> kb.add_alias(alias="Douglas", entities=["Q42", "Q463035"], probabilities=[0.6, 0.3])
> ```

| Name            | Description                                                                       |
| --------------- | --------------------------------------------------------------------------------- |
| `alias`         | The textual mention or alias. Can not be the empty string. ~~str~~                |
| `entities`      | The potential entities that the alias may refer to. ~~Iterable[Union[str, int]]~~ |
| `probabilities` | The prior probabilities of each entity. ~~Iterable[float]~~                       |

## InMemoryLookupKB.\_\_len\_\_ {#len tag="method"}

Get the total number of entities in the knowledge base.

> #### Example
>
> ```python
> total_entities = len(kb)
> ```

| Name        | Description                                           |
| ----------- | ----------------------------------------------------- |
| **RETURNS** | The number of entities in the knowledge base. ~~int~~ |

## InMemoryLookupKB.get_entity_strings {#get_entity_strings tag="method"}

Get a list of all entity IDs in the knowledge base.

> #### Example
>
> ```python
> all_entities = kb.get_entity_strings()
> ```

| Name        | Description                                               |
| ----------- | --------------------------------------------------------- |
| **RETURNS** | The list of entities in the knowledge base. ~~List[str]~~ |

## InMemoryLookupKB.get_size_aliases {#get_size_aliases tag="method"}

Get the total number of aliases in the knowledge base.

> #### Example
>
> ```python
> total_aliases = kb.get_size_aliases()
> ```

| Name        | Description                                          |
| ----------- | ---------------------------------------------------- |
| **RETURNS** | The number of aliases in the knowledge base. ~~int~~ |

## InMemoryLookupKB.get_alias_strings {#get_alias_strings tag="method"}

Get a list of all aliases in the knowledge base.

> #### Example
>
> ```python
> all_aliases = kb.get_alias_strings()
> ```

| Name        | Description                                              |
| ----------- | -------------------------------------------------------- |
| **RETURNS** | The list of aliases in the knowledge base. ~~List[str]~~ |

## InMemoryLookupKB.get_candidates {#get_candidates tag="method"}

Given a certain textual mention as input, retrieve a list of candidate entities
of type [`Candidate`](/api/kb#candidate). Wraps
[`get_alias_candidates()`](/api/kb_in_memory#get_alias_candidates).

> #### Example
>
> ```python
> from spacy.lang.en import English
> nlp = English()
> doc = nlp("Douglas Adams wrote 'The Hitchhiker's Guide to the Galaxy'.")
> candidates = kb.get_candidates(doc[0:2])
> ```

| Name        | Description                                                          |
| ----------- | -------------------------------------------------------------------- |
| `mention`   | The textual mention or alias. ~~Span~~                               |
| **RETURNS** | An iterable of relevant `Candidate` objects. ~~Iterable[Candidate]~~ |

## InMemoryLookupKB.get_candidates_batch {#get_candidates_batch tag="method"}

Same as [`get_candidates()`](/api/kb_in_memory#get_candidates), but for an
arbitrary number of mentions. The [`EntityLinker`](/api/entitylinker) component
will call `get_candidates_batch()` instead of `get_candidates()`, if the config
parameter `candidates_batch_size` is greater or equal than 1.

The default implementation of `get_candidates_batch()` executes
`get_candidates()` in a loop. We recommend implementing a more efficient way to
retrieve candidates for multiple mentions at once, if performance is of concern
to you.

> #### Example
>
> ```python
> from spacy.lang.en import English
> nlp = English()
> doc = nlp("Douglas Adams wrote 'The Hitchhiker's Guide to the Galaxy'.")
> candidates = kb.get_candidates((doc[0:2], doc[3:]))
> ```

| Name        | Description                                                                                  |
| ----------- | -------------------------------------------------------------------------------------------- |
| `mentions`  | The textual mention or alias. ~~Iterable[Span]~~                                             |
| **RETURNS** | An iterable of iterable with relevant `Candidate` objects. ~~Iterable[Iterable[Candidate]]~~ |

## InMemoryLookupKB.get_alias_candidates {#get_alias_candidates tag="method"}

Given a certain textual mention as input, retrieve a list of candidate entities
of type [`Candidate`](/api/kb#candidate).

> #### Example
>
> ```python
> candidates = kb.get_alias_candidates("Douglas")
> ```

| Name        | Description                                                   |
| ----------- | ------------------------------------------------------------- |
| `alias`     | The textual mention or alias. ~~str~~                         |
| **RETURNS** | The list of relevant `Candidate` objects. ~~List[Candidate]~~ |

## InMemoryLookupKB.get_vector {#get_vector tag="method"}

Given a certain entity ID, retrieve its pretrained entity vector.

> #### Example
>
> ```python
> vector = kb.get_vector("Q42")
> ```

| Name        | Description                          |
| ----------- | ------------------------------------ |
| `entity`    | The entity ID. ~~str~~               |
| **RETURNS** | The entity vector. ~~numpy.ndarray~~ |

## InMemoryLookupKB.get_vectors {#get_vectors tag="method"}

Same as [`get_vector()`](/api/kb_in_memory#get_vector), but for an arbitrary
number of entity IDs.

The default implementation of `get_vectors()` executes `get_vector()` in a loop.
We recommend implementing a more efficient way to retrieve vectors for multiple
entities at once, if performance is of concern to you.

> #### Example
>
> ```python
> vectors = kb.get_vectors(("Q42", "Q3107329"))
> ```

| Name        | Description                                               |
| ----------- | --------------------------------------------------------- |
| `entities`  | The entity IDs. ~~Iterable[str]~~                         |
| **RETURNS** | The entity vectors. ~~Iterable[Iterable[numpy.ndarray]]~~ |

## InMemoryLookupKB.get_prior_prob {#get_prior_prob tag="method"}

Given a certain entity ID and a certain textual mention, retrieve the prior
probability of the fact that the mention links to the entity ID.

> #### Example
>
> ```python
> probability = kb.get_prior_prob("Q42", "Douglas")
> ```

| Name        | Description                                                               |
| ----------- | ------------------------------------------------------------------------- |
| `entity`    | The entity ID. ~~str~~                                                    |
| `alias`     | The textual mention or alias. ~~str~~                                     |
| **RETURNS** | The prior probability of the `alias` referring to the `entity`. ~~float~~ |

## InMemoryLookupKB.to_disk {#to_disk tag="method"}

Save the current state of the knowledge base to a directory.

> #### Example
>
> ```python
> kb.to_disk(path)
> ```

| Name      | Description                                                                                                                                |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `path`    | A path to a directory, which will be created if it doesn't exist. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| `exclude` | List of components to exclude. ~~Iterable[str]~~                                                                                           |

## InMemoryLookupKB.from_disk {#from_disk tag="method"}

Restore the state of the knowledge base from a given directory. Note that the
[`Vocab`](/api/vocab) should also be the same as the one used to create the KB.

> #### Example
>
> ```python
> from spacy.vocab import Vocab
> vocab = Vocab().from_disk("/path/to/vocab")
> kb = FullyImplementedKB(vocab=vocab, entity_vector_length=64)
> kb.from_disk("/path/to/kb")
> ```

| Name        | Description                                                                                     |
| ----------- | ----------------------------------------------------------------------------------------------- |
| `loc`       | A path to a directory. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| `exclude`   | List of components to exclude. ~~Iterable[str]~~                                                |
| **RETURNS** | The modified `KnowledgeBase` object. ~~KnowledgeBase~~                                          |
