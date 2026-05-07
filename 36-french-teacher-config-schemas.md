# 36 - French Teacher - Configuration Schemas

## 1. Introduction to Configuration Schemas in French Language Learning Systems

The integration of technology into French language education has necessitated the development of robust, scalable, and highly configurable systems. Configuration schemas serve as the backbone of these systems, dictating how content is delivered, how learner progress is tracked, and how assessments are evaluated. In the context of French language learning, which spans from absolute beginner (A1) to mastery (C2) according to the Common European Framework of Reference for Languages (CEFR), these schemas must account for the unique linguistic features of French. This includes its complex orthography, nuanced phonology, intricate verbal morphology, and rich sociolinguistic variations.

This document provides an exhaustive reference for configuration schemas used in French language learning platforms, Learning Management Systems (LMS), and adaptive learning applications. It covers a wide array of configurations, including LMS settings, spaced repetition algorithms, CEFR mapping, assessment rubrics, pronunciation scoring, and adaptive learning paths. Each section provides detailed JSON or YAML schemas, accompanied by comprehensive explanations of the parameters and their implications for French language pedagogy.

The target audience for this document includes educational technologists, instructional designers, software engineers, and linguists involved in the development and maintenance of French language learning software. By adhering to these schemas, developers can ensure that their systems are pedagogically sound, linguistically accurate, and capable of providing a personalized and effective learning experience for students of French.

## 2. LMS Configuration for French Courses

Learning Management Systems (LMS) require specific configurations to accommodate the structure and requirements of French language courses. These configurations define the course metadata, module organization, prerequisite structures, and localization settings.

### 2.1 Course Metadata Schema

The course metadata schema defines the foundational properties of a French course. It includes parameters for the target CEFR level, the dialect of French being taught (e.g., Metropolitan French, Quebec French), and the primary pedagogical approach.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "FrenchCourseMetadata",
  "type": "object",
  "properties": {
    "courseId": {
      "type": "string",
      "description": "Unique identifier for the course."
    },
    "title": {
      "type": "string",
      "description": "The title of the course in French."
    },
    "targetCefrLevel": {
      "type": "string",
      "enum": ["A1", "A2", "B1", "B2", "C1", "C2"],
      "description": "The target CEFR level for the course."
    },
    "dialect": {
      "type": "string",
      "enum": ["fr-FR", "fr-CA", "fr-CH", "fr-BE", "fr-CD", "standard"],
      "default": "standard",
      "description": "The specific dialect or regional variant of French."
    },
    "pedagogicalApproach": {
      "type": "string",
      "enum": ["communicative", "action-oriented", "grammar-translation", "audio-lingual"],
      "default": "action-oriented",
      "description": "The primary pedagogical methodology employed."
    },
    "prerequisites": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "List of course IDs that must be completed prior to enrollment."
    },
    "estimatedHours": {
      "type": "integer",
      "minimum": 1,
      "description": "Estimated number of hours required to complete the course."
    }
  },
  "required": ["courseId", "title", "targetCefrLevel"]
}
```

### 2.2 Module Organization Schema

French courses are typically organized into modules or units, each focusing on specific communicative functions, grammatical structures, and vocabulary themes. The module organization schema dictates this structure.

```yaml
ModuleOrganizationSchema:
  type: object
  properties:
    moduleId:
      type: string
    title:
      type: string
    theme:
      type: string
      description: "Thematic focus, e.g., 'La vie quotidienne', 'Le monde du travail'."
    communicativeObjectives:
      type: array
      items:
        type: string
      description: "e.g., 'Se présenter', 'Exprimer une opinion'."
    grammarTopics:
      type: array
      items:
        type: string
      description: "e.g., 'Le passé composé', 'Le subjonctif présent'."
    vocabularyDomains:
      type: array
      items:
        type: string
      description: "e.g., 'La nourriture', 'Les voyages'."
    lessons:
      type: array
      items:
        $ref: '#/definitions/LessonSchema'
```

## 3. Spaced Repetition Algorithm Settings for French

Spaced Repetition Systems (SRS) are critical for the acquisition of French vocabulary and grammar. The algorithms driving these systems must be carefully tuned to account for the difficulty of specific items. For example, irregular verb conjugations or words with complex orthography-to-phonology mappings may require more frequent review than regular verbs or cognates.

### 3.1 SRS Configuration Schema

This schema defines the parameters for a modified SuperMemo-2 (SM-2) or similar SRS algorithm, tailored for French language items.

```json
{
  "title": "FrenchSRSConfiguration",
  "type": "object",
  "properties": {
    "algorithm": {
      "type": "string",
      "enum": ["SM-2", "FSRS", "Leitner"],
      "default": "FSRS"
    },
    "initialIntervals": {
      "type": "object",
      "properties": {
        "easy": { "type": "integer", "default": 4, "description": "Days until first review for easy items." },
        "good": { "type": "integer", "default": 1, "description": "Days until first review for good items." },
        "hard": { "type": "integer", "default": 0, "description": "Days until first review for hard items (same day)." }
      }
    },
    "easeFactorModifiers": {
      "type": "object",
      "description": "Modifiers applied to the ease factor based on linguistic features.",
      "properties": {
        "irregularVerb": { "type": "number", "default": -0.15, "description": "Penalty for irregular verbs." },
        "fauxAmi": { "type": "number", "default": -0.20, "description": "Penalty for false friends with the learner's L1." },
        "complexOrthography": { "type": "number", "default": -0.10, "description": "Penalty for words with silent letters or complex graphemes (e.g., 'oiseau')." },
        "cognate": { "type": "number", "default": 0.15, "description": "Bonus for true cognates." }
      }
    },
    "maximumInterval": {
      "type": "integer",
      "default": 36500,
      "description": "Maximum interval in days (e.g., 100 years)."
    }
  }
}
```

### 3.2 Item Difficulty Weighting

The `easeFactorModifiers` in the schema above highlight the necessity of adjusting SRS algorithms based on the linguistic properties of French. A word like *développement* might have a standard ease factor, but a false friend like *actuellement* (currently, not actually) requires a penalty to ensure it is reviewed more frequently until the learner overcomes the L1 interference. Similarly, highly irregular verbs like *aller* or *être* in the subjunctive mood demand more rigorous scheduling.

## 4. CEFR Level Mapping Schemas

The CEFR provides a standardized framework for describing language proficiency. In digital learning systems, content, assessments, and learner profiles must be mapped to these levels. The mapping schemas define the criteria and thresholds for each level.

### 4.1 CEFR Competency Mapping

This schema maps specific linguistic competencies to CEFR levels, allowing the system to tag content appropriately and evaluate learner proficiency across different skills (reading, writing, listening, speaking).

```yaml
CEFRMappingSchema:
  type: object
  properties:
    level:
      type: string
      enum: [A1, A2, B1, B2, C1, C2]
    globalScale:
      type: string
      description: "The global description of the level according to the Council of Europe."
    competencies:
      type: object
      properties:
        listening:
          type: array
          items:
            type: string
            description: "e.g., 'Can understand familiar words and very basic phrases concerning myself, my family and immediate concrete surroundings when people speak slowly and clearly.' (A1)"
        reading:
          type: array
          items:
            type: string
        spokenInteraction:
          type: array
          items:
            type: string
        spokenProduction:
          type: array
          items:
            type: string
        writing:
          type: array
          items:
            type: string
    grammaticalStructures:
      type: array
      items:
        type: string
      description: "Grammar expected at this level. e.g., for B1: 'Conditionnel présent', 'Pronoms relatifs simples (qui, que, où, dont)'."
    lexicalRange:
      type: integer
      description: "Estimated active vocabulary size required for this level."
```

### 4.2 Content Tagging and Retrieval

By utilizing the `CEFRMappingSchema`, content management systems can automatically filter and retrieve reading passages, audio clips, and exercises that match a learner's current proficiency level. For instance, a text tagged with `grammaticalStructures` including the *passé simple* would automatically be restricted to B2 or higher levels, as this literary tense is not typically introduced earlier.

## 5. Assessment Rubric Configurations

Automated and semi-automated assessments require highly structured rubrics to ensure objective and consistent grading. For French, these rubrics must evaluate not only grammatical accuracy and vocabulary usage but also sociolinguistic appropriateness (e.g., the *tu* vs. *vous* distinction) and pragmatic competence.

### 5.1 Writing Assessment Rubric Schema

This schema defines the parameters for evaluating written French production, such as essays or emails.

```json
{
  "title": "FrenchWritingRubric",
  "type": "object",
  "properties": {
    "rubricId": { "type": "string" },
    "taskType": {
      "type": "string",
      "enum": ["email_informal", "email_formal", "essay", "summary", "creative_writing"]
    },
    "criteria": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "weight": { "type": "number", "minimum": 0, "maximum": 1 },
          "levels": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "score": { "type": "integer" },
                "description": { "type": "string" }
              }
            }
          }
        }
      }
    }
  }
}
```

### 5.2 Example Configuration for B1 Formal Email

| Criterion | Weight | Score 1 (Poor) | Score 3 (Adequate) | Score 5 (Excellent) |
| :--- | :--- | :--- | :--- | :--- |
| **Task Achievement** | 0.30 | Fails to address the prompt. | Addresses the main points but lacks detail. | Fully addresses all points with appropriate elaboration. |
| **Grammatical Accuracy** | 0.25 | Frequent errors impede communication. | Some errors, but meaning is generally clear. | High degree of control; minor errors do not impede meaning. |
| **Lexical Resource** | 0.20 | Limited vocabulary; frequent repetition. | Sufficient vocabulary to express ideas, some circumlocution. | Broad range of vocabulary; precise word choice. |
| **Sociolinguistic Appropriateness** | 0.15 | Inappropriate register (e.g., uses *tu* instead of *vous*). | Generally appropriate register, minor lapses. | Consistently appropriate formal register; uses standard formulas (*formules de politesse*). |
| **Coherence & Cohesion** | 0.10 | Disjointed sentences; lack of connectors. | Uses basic connectors (*mais*, *parce que*). | Effective use of a variety of cohesive devices (*cependant*, *en outre*). |

## 6. Pronunciation Scoring Parameters

Automated speech recognition (ASR) and pronunciation scoring are vital components of modern language learning applications. For French, the scoring parameters must be highly sensitive to specific phonological features, such as nasal vowels, the uvular 'r' (/ʁ/), liaisons, and elisions.

### 6.1 Pronunciation Evaluation Schema

This schema configures the parameters for an ASR engine evaluating French speech.

```yaml
PronunciationScoringSchema:
  type: object
  properties:
    engineId:
      type: string
    targetDialect:
      type: string
      default: "fr-FR"
    scoringWeights:
      type: object
      properties:
        phonemicAccuracy:
          type: number
          default: 0.40
          description: "Accuracy of individual phonemes."
        prosody:
          type: number
          default: 0.20
          description: "Intonation and rhythm."
        fluency:
          type: number
          default: 0.20
          description: "Speech rate and pausing."
        liaisonExecution:
          type: number
          default: 0.10
          description: "Correct application of mandatory and optional liaisons."
        elisionExecution:
          type: number
          default: 0.10
          description: "Correct application of elisions (e.g., 'l'arbre' instead of 'le arbre')."
    phonemeSpecificTolerances:
      type: array
      items:
        type: object
        properties:
          phoneme:
            type: string
            description: "IPA symbol, e.g., 'ʁ', 'œ̃', 'y'."
          toleranceLevel:
            type: string
            enum: [strict, moderate, lenient]
          l1InterferenceMapping:
            type: array
            items:
              type: string
            description: "Common L1 substitutions to detect (e.g., English speakers substituting /u/ for /y/)."
```

### 6.2 The Importance of Liaison and Elision

Unlike many other languages, French pronunciation is heavily dependent on the interaction between adjacent words. The `liaisonExecution` and `elisionExecution` parameters are crucial. A system must be configured to penalize the failure to make a mandatory liaison (e.g., *les amis* pronounced without the /z/ sound) or the incorrect application of a forbidden liaison (e.g., *et // il* pronounced with a /t/ sound). Furthermore, the `phonemeSpecificTolerances` allow the system to be more forgiving of difficult phonemes (like the nasal /œ̃/ which is merging with /ɛ̃/ in modern Parisian French) while remaining strict on phonemic distinctions that alter meaning (e.g., /u/ vs. /y/ in *vous* vs. *vu*).

## 7. Adaptive Learning Path Configurations

Adaptive learning systems dynamically adjust the sequence and difficulty of content based on learner performance. The configuration schemas for these paths define the rules for progression, remediation, and content selection.

### 7.1 Adaptive Path Schema

This schema outlines the logic for an adaptive learning engine navigating a learner through a French curriculum.

```json
{
  "title": "FrenchAdaptivePath",
  "type": "object",
  "properties": {
    "pathId": { "type": "string" },
    "entryPoint": {
      "type": "object",
      "properties": {
        "assessmentId": { "type": "string" },
        "placementRules": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "scoreRange": { "type": "array", "items": { "type": "integer" } },
              "startingModuleId": { "type": "string" }
            }
          }
        }
      }
    },
    "progressionRules": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "condition": {
            "type": "string",
            "description": "e.g., 'module_score >= 80%'"
          },
          "action": {
            "type": "string",
            "enum": ["advance", "remediate", "enrich"]
          },
          "targetId": {
            "type": "string",
            "description": "The ID of the next module, remediation exercise, or enrichment activity."
          }
        }
      }
    },
    "remediationStrategies": {
      "type": "object",
      "properties": {
        "grammarError": {
          "type": "string",
          "description": "Action to take upon repeated grammar errors (e.g., 'present_explicit_rule_then_drill')."
        },
        "vocabularyError": {
          "type": "string",
          "description": "Action to take upon vocabulary errors (e.g., 'increase_srs_frequency')."
        }
      }
    }
  }
}
```

### 7.2 Diagnostic Remediation

In the context of French, the `remediationStrategies` must be highly granular. If a learner consistently fails exercises involving the *passé composé* vs. *imparfait* distinction, the system should not merely present more of the same exercises. Instead, the configuration should trigger a remediation path that breaks down the underlying concepts: first testing the conjugation of auxiliary verbs (*avoir* vs. *être*), then testing past participle agreement, and finally addressing the semantic distinction between completed actions and ongoing states in the past.

## 8. Data Models and API Schemas for French Language Content

To ensure interoperability between different components of a language learning ecosystem (e.g., a mobile app, a web dashboard, and a backend analytics server), standardized data models and API schemas are required.

### 8.1 Vocabulary Item Schema

This schema defines how a single French vocabulary item is represented in the database and transmitted via APIs. It includes essential linguistic metadata.

```yaml
VocabularyItemSchema:
  type: object
  properties:
    itemId:
      type: string
    lemma:
      type: string
      description: "The base form of the word (e.g., 'manger', 'beau')."
    partOfSpeech:
      type: string
      enum: [noun, verb, adjective, adverb, pronoun, preposition, conjunction, interjection]
    gender:
      type: string
      enum: [masculine, feminine, neuter, none]
      description: "Crucial for French nouns and adjectives."
    number:
      type: string
      enum: [singular, plural, invariable]
    cefrLevel:
      type: string
      enum: [A1, A2, B1, B2, C1, C2]
    translations:
      type: array
      items:
        type: object
        properties:
          languageCode: { type: "string" }
          translation: { type: "string" }
    exampleSentences:
      type: array
      items:
        type: object
        properties:
          french: { type: "string" }
          translation: { type: "string" }
          audioUrl: { type: "string", format: "uri" }
    audioPronunciationUrl:
      type: string
      format: "uri"
    ipaTranscription:
      type: string
      description: "International Phonetic Alphabet transcription."
```

### 8.2 Grammar Rule Schema

Representing grammar rules as structured data allows the system to dynamically generate explanations and link errors to specific rules.

```json
{
  "title": "GrammarRuleSchema",
  "type": "object",
  "properties": {
    "ruleId": { "type": "string" },
    "title": { "type": "string", "description": "e.g., 'Agreement of the past participle with avoir'" },
    "description": { "type": "string" },
    "cefrLevel": { "type": "string" },
    "conditions": {
      "type": "array",
      "items": { "type": "string" },
      "description": "e.g., ['Auxiliary is avoir', 'Direct object precedes the verb']"
    },
    "exceptions": {
      "type": "array",
      "items": { "type": "string" },
      "description": "e.g., ['En as a direct object pronoun']"
    },
    "examples": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "correct": { "type": "string" },
          "incorrect": { "type": "string" },
          "explanation": { "type": "string" }
        }
      }
    }
  }
}
```

## 9. Conclusion

The configuration schemas detailed in this document provide a comprehensive framework for building sophisticated, effective, and linguistically accurate French language learning systems. By explicitly modeling the complexities of the French language—from its phonological nuances and strict grammatical rules to its CEFR-aligned progression—developers can create platforms that truly adapt to the needs of the learner.

Whether configuring an LMS for a university course, tuning the spaced repetition algorithm for a mobile vocabulary app, or setting the parameters for an automated pronunciation scoring engine, adherence to these structured schemas ensures consistency, scalability, and pedagogical efficacy. As language learning technology continues to evolve, these schemas will serve as a vital foundation for the next generation of intelligent tutoring systems and immersive digital environments for French acquisition.

## References

[1] Council of Europe. (2001). Common European Framework of Reference for Languages: Learning, teaching, assessment. Cambridge University Press.
[2] Wozniak, P. A. (1990). Optimization of learning. Master's thesis, University of Technology in Poznan.
[3] Eskenazi, M. (2009). An overview of spoken language technology for education. Speech Communication, 51(10), 832-844.
[4] Lyster, R., & Ranta, L. (1997). Error treatment and learner uptake in communicative classrooms. Studies in second language acquisition, 19(1), 37-66.
