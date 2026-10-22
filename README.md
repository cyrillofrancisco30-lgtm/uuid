RFC-0001: XA-TRUST-GOV-001 — Global Claim Non-Derivability Policy
Status: Standards Track
Data: 15 de setembro de 2026
Domínio: Trust & Governance Framework (XA-TRUST)
Autor: Architecture & Trust Governance Working Group
1. Resumo Executivo (Abstract)
Este documento especifica formalmente a regra de governança XA-TRUST-GOV-001 (Global Claim Non-Derivability). A regra proíbe estritamente a inferência automática do status GLOBAL VERIFIED a partir da conjunção de status VERIFIED de escopo local (A \land B \land C). Define-se a esteira obrigatória de reavaliação de segunda ordem, os cenários de retenção em NOT_PROMOTABLE, as estruturas dos payloads de resposta e os contratos formais em JSON-Schema.
2. Semântica e Formalização Matemática
2.1 Teorema da Não-Derivabilidade Global
A aprovação no escopo de uma alegação local (claim) certifica a exatidão dos dados exclusivamente dentro da sua jurisdição e limites operacionais. A conjunção de alegações locais válidas atua apenas como evidência de entrada (evidence input) para a decisão macro, não configurando transitividade de decisão.
2.2 Fluxo Sequencial de Decisão
LOCAL VERIFIED CLAIMS (A, B, C)
        │
        ▼
GLOBAL EVALUATION INPUTS
        │
        ▼
GLOBAL RE-EVALUATION
        ├── 1. TARGET POPULATION
        ├── 2. COVERAGE
        ├── 3. COMPLETENESS
        ├── 4. AGGREGATION POLICY
        └── 5. AGGREGATION VERIFICATION
        │
        ▼
GLOBAL CLAIM DECISION
        ├── VERIFIED (Sucesso)
        └── NOT_PROMOTABLE (Bloqueio)

3. Critérios de Reavaliação Global e Bloqueio
A transição para GLOBAL VERIFIED exige a satisfação cumulativa dos cinco pilares de governança. O não atendimento de qualquer um dos critérios força o estado padrão de salvaguarda NOT_PROMOTABLE.
 * Target Population: A amostragem combinada das alegações locais deve cobrir exatamente o universo e o domínio representativo exigidos pela norma macro.
 * Coverage: O percentual de cobertura efetivo (\text{coverage\_ratio}) deve atingir o limiar exigido (\text{threshold\_required}, tipicamente 1.00).
 * Completeness: Ausência de lacunas ocultas de dados e garantia de alinhamento temporal e geográfico síncrono entre as fontes.
 * Aggregation Policy: Aplicação rigorosa das regras metodológicas de combinação (ex: médias ponderadas em vez de médias simples).
 * Aggregation Verification: Checagem de não-linearidades, deduplicação cross-claim e geração da prova criptográfica das evidências agregadas.
4. Modelos de Payload JSON
4.1 Resposta de Rejeição / Bloqueio (NOT_PROMOTABLE)
{
  "claim_id": "claim-global-98421-2026",
  "rule_id": "XA-TRUST-GOV-001",
  "decision": "NOT_PROMOTABLE",
  "evaluated_at": "2026-09-15T11:42:00Z",
  "evaluation_context": {
    "scope": "GLOBAL",
    "target_population": "LATAM_ALL_SEGMENTS",
    "local_inputs": [
      { "claim_id": "claim-local-A", "status": "VERIFIED" },
      { "claim_id": "claim-local-B", "status": "VERIFIED" },
      { "claim_id": "claim-local-C", "status": "VERIFIED" }
    ]
  },
  "blocking_reasons": [
    {
      "code": "ERR_SCOPE_UNDERCOVERAGE",
      "category": "COVERAGE",
      "severity": "CRITICAL",
      "message": "Local claims do not satisfy global target population requirements.",
      "details": {
        "required_regions": ["SUL", "SUDESTE", "CENTRO-OESTE", "NORTE", "NORDESTE"],
        "covered_regions": ["SUL", "SUDESTE", "CENTRO-OESTE"],
        "missing_regions": ["NORTE", "NORDESTE"],
        "coverage_ratio": 0.60,
        "threshold_required": 1.00
      }
    }
  ],
  "audit_metadata": {
    "evaluator_engine": "trust-gov-evaluator-v2.4.0",
    "policy_version": "1.8.0",
    "execution_id": "exec-uuid-77123-99a",
    "trace_id": "0af7651916cd43dd8448eb211c80319c",
    "sha256_input_proof": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
  }
}

4.2 Resposta de Sucesso (GLOBAL VERIFIED)
{
  "claim_id": "claim-global-98421-2026",
  "rule_id": "XA-TRUST-GOV-001",
  "decision": "VERIFIED",
  "evaluated_at": "2026-09-15T11:45:00Z",
  "evaluation_context": {
    "scope": "GLOBAL",
    "target_population": "LATAM_ALL_SEGMENTS",
    "local_inputs": [
      { "claim_id": "claim-local-A", "status": "VERIFIED" },
      { "claim_id": "claim-local-B", "status": "VERIFIED" },
      { "claim_id": "claim-local-C", "status": "VERIFIED" }
    ]
  },
  "verification_proof": {
    "coverage_satisfied": {
      "coverage_ratio": 1.00,
      "threshold_required": 1.00
    },
    "completeness_satisfied": true,
    "aggregation_policy_applied": "WEIGHTED_AVERAGE_BY_VOLUME",
    "aggregation_verification": {
      "verified": true,
      "merkle_root": "8f4e2a3b1c9d8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f"
    }
  },
  "audit_metadata": {
    "evaluator_engine": "trust-gov-evaluator-v2.4.0",
    "policy_version": "1.8.0",
    "execution_id": "exec-uuid-77124-00b",
    "trace_id": "1bf7651916cd43dd8448eb211c80320d",
    "sha256_input_proof": "a4c0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852c966"
  }
}

5. Especificações JSON-Schema (Draft 2020-12)
5.1 Schema para NOT_PROMOTABLE
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://gov.trust.xa/schemas/v1/not-promotable-decision.schema.json",
  "title": "NotPromotableDecisionPayload",
  "type": "object",
  "required": ["claim_id", "rule_id", "decision", "evaluated_at", "evaluation_context", "blocking_reasons", "audit_metadata"],
  "properties": {
    "claim_id": { "type": "string" },
    "rule_id": { "type": "string", "const": "XA-TRUST-GOV-001" },
    "decision": { "type": "string", "enum": ["NOT_PROMOTABLE"] },
    "evaluated_at": { "type": "string", "format": "date-time" },
    "evaluation_context": {
      "type": "object",
      "required": ["scope", "target_population", "local_inputs"],
      "properties": {
        "scope": { "type": "string", "enum": ["GLOBAL"] },
        "target_population": { "type": "string" },
        "local_inputs": {
          "type": "array",
          "minItems": 1,
          "items": {
            "type": "object",
            "required": ["claim_id", "status"],
            "properties": {
              "claim_id": { "type": "string" },
              "status": { "type": "string", "enum": ["VERIFIED", "NOT_VERIFIED", "REVOKED"] }
            },
            "additionalProperties": false
          }
        }
      },
      "additionalProperties": false
    },
    "blocking_reasons": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "required": ["code", "category", "severity", "message", "details"],
        "properties": {
          "code": { "type": "string", "pattern": "^ERR_[A-Z0-9_]+$" },
          "category": { "type": "string", "enum": ["TARGET_POPULATION", "COVERAGE", "COMPLETENESS", "AGGREGATION_POLICY", "AGGREGATION_VERIFICATION"] },
          "severity": { "type": "string", "enum": ["LOW", "MEDIUM", "HIGH", "CRITICAL"] },
          "message": { "type": "string" },
          "details": { "type": "object", "additionalProperties": true }
        },
        "additionalProperties": false
      }
    },
    "audit_metadata": {
      "type": "object",
      "required": ["evaluator_engine", "policy_version", "execution_id", "trace_id", "sha256_input_proof"],
      "properties": {
        "evaluator_engine": { "type": "string" },
        "policy_version": { "type": "string" },
        "execution_id": { "type": "string" },
        "trace_id": { "type": "string" },
        "sha256_input_proof": { "type": "string", "pattern": "^[a-fA-F0-9]{64}$" }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}

5.2 Schema para GLOBAL VERIFIED
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://gov.trust.xa/schemas/v1/global-verified-decision.schema.json",
  "title": "GlobalVerifiedDecisionPayload",
  "type": "object",
  "required": ["claim_id", "rule_id", "decision", "evaluated_at", "evaluation_context", "verification_proof", "audit_metadata"],
  "properties": {
    "claim_id": { "type": "string" },
    "rule_id": { "type": "string", "const": "XA-TRUST-GOV-001" },
    "decision": { "type": "string", "enum": ["VERIFIED"] },
    "evaluated_at": { "type": "string", "format": "date-time" },
    "evaluation_context": {
      "type": "object",
      "required": ["scope", "target_population", "local_inputs"],
      "properties": {
        "scope": { "type": "string", "enum": ["GLOBAL"] },
        "target_population": { "type": "string" },
        "local_inputs": {
          "type": "array",
          "minItems": 1,
          "items": {
            "type": "object",
            "required": ["claim_id", "status"],
            "properties": {
              "claim_id": { "type": "string" },
              "status": { "type": "string", "enum": ["VERIFIED"] }
            },
            "additionalProperties": false
          }
        }
      },
      "additionalProperties": false
    },
    "verification_proof": {
      "type": "object",
      "required": ["coverage_satisfied", "completeness_satisfied", "aggregation_policy_applied", "aggregation_verification"],
      "properties": {
        "coverage_satisfied": {
          "type": "object",
          "required": ["coverage_ratio", "threshold_required"],
          "properties": {
            "coverage_ratio": { "type": "number", "minimum": 0.0, "maximum": 1.0 },
            "threshold_required": { "type": "number", "minimum": 0.0, "maximum": 1.0 }
          },
          "additionalProperties": false
        },
        "completeness_satisfied": { "type": "boolean", "const": true },
        "aggregation_policy_applied": { "type": "string" },
        "aggregation_verification": {
          "type": "object",
          "required": ["verified", "merkle_root"],
          "properties": {
            "verified": { "type": "boolean", "const": true },
            "merkle_root": { "type": "string", "pattern": "^[a-fA-F0-9]{64}$" }
          },
          "additionalProperties": false
        }
      },
      "additionalProperties": false
    },
    "audit_metadata": {
      "type": "object",
      "required": ["evaluator_engine", "policy_version", "execution_id", "trace_id", "sha256_input_proof"],
      "properties": {
        "evaluator_engine": { "type": "string" },
        "policy_version": { "type": "string" },
        "execution_id": { "type": "string" },
        "trace_id": { "type": "string" },
        "sha256_input_proof": { "type": "string", "pattern": "^[a-fA-F0-9]{64}$" }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}

6. Considerações de Segurança e Conformidade
 * Invariabilidade de Escopo: Motores de decisão downstream devem rejeitar payloads globais que não contenham o atributo sha256_input_proof devidamente assinado e rastreável.
 * Defesa em Profundidade: A presença de additionalProperties: false impede ataques de sobrescrita de estado e injeção de parâmetros não homologados durante a transmissão de mensagens no barramento.


Aqui está a tabela formal de elementos e papéis na governança XA-TRUST:
| Elemento | Papel / Categoria de Evidência |
|---|---|
| Security FAQ | SOURCE_ARTIFACT |
| ZDR documentation | POLICY_EVIDENCE |
| Team ZDR Active | CONFIGURATION_EVIDENCE |
| API Request | REQUEST_ARTIFACT |
| Execução concreta | E3 EXECUTION_EVENT_EVIDENCE |
| API Response | RESPONSE_ARTIFACT |
| x-zero-data-retention=true | EXECUTION_CONTEXT_EVIDENCE |
| Assertion header == true | TEST_ASSERTION |
| Observed/Expected/Pass | E4 TEST_RESULT_EVIDENCE |
| Hash/signature/binding válidos | E5 CRYPTOGRAPHIC_BINDING_EVIDENCE |
| Verificador independente | E6 INDEPENDENT_VERIFICATION_EVENT |
| Claim específica | E7 CLAIM-SCOPED VERIFIED |
| FAQ → VERIFIED global | PROIBIDO (XA-TRUST-GOV-001) |
Regra de Governança Aplicada
 * PROIBIDO (Non-Derivability): Tentar derivar a validação global a partir de um artefato isolado como o Security FAQ viola diretamente a regra XA-TRUST-GOV-001.
 * Cadeia de Evidências: A elevação para GLOBAL VERIFIED exige a avaliação integrada de toda a esteira de evidências (E_3 \to E_7), garantindo que alegações de escopo local (CLAIM-SCOPED VERIFIED) não forcem uma promoção global automática sem a devida reavaliação de escopo, cobertura e integridade criptográfica.



<!--
  -- This file is auto-generated from README_js.md. Changes should be made there.
  -->

# uuid [![CI](https://github.com/uuidjs/uuid/workflows/CI/badge.svg)](https://github.com/uuidjs/uuid/actions?query=workflow%3ACI) [![Browser](https://github.com/uuidjs/uuid/workflows/Browser/badge.svg)](https://github.com/uuidjs/uuid/actions/workflows/browser.yml)

For the creation of [RFC9562](https://www.rfc-editor.org/rfc/rfc9562.html) (formerly [RFC4122](https://www.rfc-editor.org/rfc/rfc4122.html)) UUIDs

- **Complete** - Support for all RFC9562 UUID versions
- **Cross-platform** - Support for...
  - [Typescript](#support)
  - [Chrome, Safari, Firefox, and Edge](#support)
  - [NodeJS](#support)
  - [React Native / Expo](#react-native--expo)
- **Secure** - Uses modern `crypto` API for random values
- **Compact** - Zero-dependency, [tree-shakable](https://developer.mozilla.org/en-US/docs/Glossary/Tree_shaking)
- **CLI** - [`uuid` command line](#command-line) utility

<!-- prettier-ignore -->
> [!NOTE]
>
> Starting with `uuid@12` CommonJS is no longer supported.  See [implications](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c) and [motivation](https://github.com/uuidjs/uuid/issues/881) for details.

## Quickstart

**1. Install**

```shell
npm install uuid
```

**2. Create a UUID**

```javascript
import { v4 as uuidv4 } from 'uuid';

uuidv4(); // ⇨ 'b18794e8-5d0d-417c-b361-ba38e78411b4'
```

For timestamp UUIDs, namespace UUIDs, and other options read on ...

## API Summary

|  |  |
| --- | --- |
| [`uuid.NIL`](#uuidnil) | The nil UUID string (all zeros) |
| [`uuid.MAX`](#uuidmax) | The max UUID string (all ones) |
| [`uuid.parse()`](#uuidparsestr) | Convert UUID string to array of bytes |
| [`uuid.stringify()`](#uuidstringifyarr-offset) | Convert array of bytes to UUID string |
| [`uuid.v1()`](#uuidv1options-buffer-offset) | Generate a version 1 (timestamp) UUID |
| [`uuid.v1ToV6()`](#uuidv1tov6uuid) | Convert a version 1 UUID to version 6 |
| [`uuid.v3()`](#uuidv3name-namespace-buffer-offset) | Generate a version 3 (namespace w/ MD5) UUID |
| [`uuid.v4()`](#uuidv4options-buffer-offset) | Generate a version 4 (random) UUID |
| [`uuid.v5()`](#uuidv5name-namespace-buffer-offset) | Generate a version 5 (namespace w/ SHA-1) UUID |
| [`uuid.v6()`](#uuidv6options-buffer-offset) | Generate a version 6 (timestamp, reordered) UUID |
| [`uuid.v6ToV1()`](#uuidv6tov1uuid) | Convert a version 6 UUID to version 1 |
| [`uuid.v7()`](#uuidv7options-buffer-offset) | Generate a version 7 (Unix Epoch time-based) UUID |
| ~~[`uuid.v8()`](#uuidv8)~~ | "Intentionally left blank" |
| [`uuid.validate()`](#uuidvalidatestr) | Test a string to see if it is a valid UUID |
| [`uuid.version()`](#uuidversionstr) | Detect RFC version of a UUID |

## API

### uuid.NIL

The nil UUID string (all zeros).

Example:

```javascript
import { NIL as NIL_UUID } from 'uuid';

NIL_UUID; // ⇨ '00000000-0000-0000-0000-000000000000'
```

### uuid.MAX

The max UUID string (all ones).

Example:

```javascript
import { MAX as MAX_UUID } from 'uuid';

MAX_UUID; // ⇨ 'ffffffff-ffff-ffff-ffff-ffffffffffff'
```

### uuid.parse(str)

Convert UUID string to array of bytes

|           |                                          |
| --------- | ---------------------------------------- |
| `str`     | A valid UUID `String`                    |
| _returns_ | `Uint8Array[16]`                         |
| _throws_  | `TypeError` if `str` is not a valid UUID |

<!-- prettier-ignore -->
> [!NOTE]
> Ordering of values in the byte arrays used by `parse()` and `stringify()` follows the left &Rarr; right order of hex-pairs in UUID strings. As shown in the example below.

Example:

```javascript
import { parse as uuidParse } from 'uuid';

// Parse a UUID
uuidParse('6ec0bd7f-11c0-43da-975e-2a8ad9ebae0b'); // ⇨
// Uint8Array(16) [
//   110, 192, 189, 127,  17,
//   192,  67, 218, 151,  94,
//    42, 138, 217, 235, 174,
//    11
// ]
```

### uuid.stringify(arr[, offset])

Convert array of bytes to UUID string

|                |                                                                              |
| -------------- | ---------------------------------------------------------------------------- |
| `arr`          | `Array`-like collection of 16 values (starting from `offset`) between 0-255. |
| [`offset` = 0] | `Number` Starting index in the Array                                         |
| _returns_      | `String`                                                                     |
| _throws_       | `TypeError` if a valid UUID string cannot be generated                       |

<!-- prettier-ignore -->
> [!NOTE]
> Ordering of values in the byte arrays used by `parse()` and `stringify()` follows the left &Rarr; right order of hex-pairs in UUID strings. As shown in the example below.

Example:

```javascript
import { stringify as uuidStringify } from 'uuid';

const uuidBytes = Uint8Array.of(
  0x6e,
  0xc0,
  0xbd,
  0x7f,
  0x11,
  0xc0,
  0x43,
  0xda,
  0x97,
  0x5e,
  0x2a,
  0x8a,
  0xd9,
  0xeb,
  0xae,
  0x0b
);

uuidStringify(uuidBytes); // ⇨ '6ec0bd7f-11c0-43da-975e-2a8ad9ebae0b'
```

### uuid.v1([options[, buffer[, offset]]])

Create an RFC version 1 (timestamp) UUID

|  |  |
| --- | --- |
| [`options`] | `Object` with one or more of the following properties: |
| [`options.node = (random)` ] | RFC "node" field as an `Array[6]` of byte values (per 4.1.6) |
| [`options.clockseq = (random)`] | RFC "clock sequence" as a `Number` between 0 - 0x3fff |
| [`options.msecs = (current time)`] | RFC "timestamp" field (`Number` of milliseconds, unix epoch) |
| [`options.nsecs = 0`] | RFC "timestamp" field (`Number` of nanoseconds to add to `msecs`, should be 0-10,000) |
| [`options.random = (random)`] | `Array` of 16 random bytes (0-255) used to generate other fields, above |
| [`options.rng`] | Alternative to `options.random`, a `Function` that returns an `Array` of 16 random bytes (0-255) |
| [`buffer`] | `Uint8Array` or `Uint8Array` subtype (e.g. Node.js `Buffer`). If provided, binary UUID is written into the array, starting at `offset` |
| [`offset` = 0] | `Number` Index to start writing UUID bytes in `buffer` |
| _returns_ | UUID `String` if no `buffer` is specified, otherwise returns `buffer` |
| _throws_ | `Error` if more than 10M UUIDs/sec are requested |

Example:

```javascript
import { v1 as uuidv1 } from 'uuid';

uuidv1(); // ⇨ '57fd0000-c7d3-11ef-841d-514d2167fc5b'
```

Example using `options`:

```javascript
import { v1 as uuidv1 } from 'uuid';

const options = {
  node: Uint8Array.of(0x01, 0x23, 0x45, 0x67, 0x89, 0xab),
  clockseq: 0x1234,
  msecs: new Date('2011-11-01').getTime(),
  nsecs: 5678,
};
uuidv1(options); // ⇨ '710b962e-041c-11e1-9234-0123456789ab'
```

### uuid.v1ToV6(uuid)

Convert a UUID from version 1 to version 6

```javascript
import { v1ToV6 } from 'uuid';

v1ToV6('92f62d9e-22c4-11ef-97e9-325096b39f47'); // ⇨ '1ef22c49-2f62-6d9e-97e9-325096b39f47'
```

### uuid.v3(name, namespace[, buffer[, offset]])

Create an RFC version 3 (namespace w/ MD5) UUID

API is identical to `v5()`, but uses "v3" instead.

<!-- prettier-ignore -->
> [!IMPORTANT]
> Per the RFC, "_If backward compatibility is not an issue, SHA-1 [Version 5] is preferred_."

### uuid.v4([options[, buffer[, offset]]])

Create an RFC version 4 (random) UUID

|  |  |
| --- | --- |
| [`options`] | `Object` with one or more of the following properties: |
| [`options.random`] | `Array` of 16 random bytes (0-255) |
| [`options.rng`] | Alternative to `options.random`, a `Function` that returns an `Array` of 16 random bytes (0-255) |
| [`buffer`] | `Uint8Array` or `Uint8Array` subtype (e.g. Node.js `Buffer`). If provided, binary UUID is written into the array, starting at `offset` |
| [`offset` = 0] | `Number` Index to start writing UUID bytes in `buffer` |
| _returns_ | UUID `String` if no `buffer` is specified, otherwise returns `buffer` |

Example:

```javascript
import { v4 as uuidv4 } from 'uuid';

uuidv4(); // ⇨ 'b18794e8-5d0d-417c-b361-ba38e78411b4'
```

Example using predefined `random` values:

```javascript
import { v4 as uuidv4 } from 'uuid';

const v4options = {
  random: Uint8Array.of(
    0x10,
    0x91,
    0x56,
    0xbe,
    0xc4,
    0xfb,
    0xc1,
    0xea,
    0x71,
    0xb4,
    0xef,
    0xe1,
    0x67,
    0x1c,
    0x58,
    0x36
  ),
};
uuidv4(v4options); // ⇨ '109156be-c4fb-41ea-b1b4-efe1671c5836'
```

### uuid.v5(name, namespace[, buffer[, offset]])

Create an RFC version 5 (namespace w/ SHA-1) UUID

|  |  |
| --- | --- |
| `name` | `String \| Array` |
| `namespace` | `String \| Array[16]` Namespace UUID |
| [`buffer`] | `Uint8Array` or `Uint8Array` subtype (e.g. Node.js `Buffer`). If provided, binary UUID is written into the array, starting at `offset` |
| [`offset` = 0] | `Number` Index to start writing UUID bytes in `buffer` |
| _returns_ | UUID `String` if no `buffer` is specified, otherwise returns `buffer` |

<!-- prettier-ignore -->
> [!NOTE]
> The RFC `DNS` and `URL` namespaces are available as `v5.DNS` and `v5.URL`.

Example with custom namespace:

```javascript
import { v5 as uuidv5 } from 'uuid';

// Define a custom namespace.  Readers, create your own using something like
// https://www.uuidgenerator.net/
const MY_NAMESPACE = '1b671a64-40d5-491e-99b0-da01ff1f3341';

uuidv5('Hello, World!', MY_NAMESPACE); // ⇨ '630eb68f-e0fa-5ecc-887a-7c7a62614681'
```

Example with RFC `URL` namespace:

```javascript
import { v5 as uuidv5 } from 'uuid';

uuidv5('https://www.w3.org/', uuidv5.URL); // ⇨ 'c106a26a-21bb-5538-8bf2-57095d1976c1'
```

### uuid.v6([options[, buffer[, offset]]])

Create an RFC version 6 (timestamp, reordered) UUID

This method takes the same arguments as uuid.v1().

```javascript
import { v6 as uuidv6 } from 'uuid';

uuidv6(); // ⇨ '1efc7d35-7fd0-6000-841d-504d2167fc5b'
```

Example using `options`:

```javascript
import { v6 as uuidv6 } from 'uuid';

const options = {
  node: [0x01, 0x23, 0x45, 0x67, 0x89, 0xab],
  clockseq: 0x1234,
  msecs: new Date('2011-11-01').getTime(),
  nsecs: 5678,
};
uuidv6(options); // ⇨ '1e1041c7-10b9-662e-9234-0123456789ab'
```

### uuid.v6ToV1(uuid)

Convert a UUID from version 6 to version 1

```javascript
import { v6ToV1 } from 'uuid';

v6ToV1('1ef22c49-2f62-6d9e-97e9-325096b39f47'); // ⇨ '92f62d9e-22c4-11ef-97e9-325096b39f47'
```

### uuid.v7([options[, buffer[, offset]]])

Create an RFC version 7 (random) UUID

|  |  |
| --- | --- |
| [`options`] | `Object` with one or more of the following properties: |
| [`options.msecs = (current time)`] | RFC "timestamp" field (`Number` of milliseconds, unix epoch) |
| [`options.random = (random)`] | `Array` of 16 random bytes (0-255) used to generate other fields, above |
| [`options.rng`] | Alternative to `options.random`, a `Function` that returns an `Array` of 16 random bytes (0-255) |
| [`options.seq = (random)`] | 32-bit sequence `Number` between 0 - 0xffffffff. This may be provided to help ensure uniqueness for UUIDs generated within the same millisecond time interval. Default = random value. |
| [`buffer`] | `Uint8Array` or `Uint8Array` subtype (e.g. Node.js `Buffer`). If provided, binary UUID is written into the array, starting at `offset` |
| [`offset` = 0] | `Number` Index to start writing UUID bytes in `buffer` |
| _returns_ | UUID `String` if no `buffer` is specified, otherwise returns `buffer` |

Example:

```javascript
import { v7 as uuidv7 } from 'uuid';

uuidv7(); // ⇨ '01941f29-7c00-73e4-a310-744d2167fc5b'
```

### ~~uuid.v8()~~

**_"Intentionally left blank"_**

<!-- prettier-ignore -->
> [!NOTE]
> Version 8 (experimental) UUIDs are "[for experimental or vendor-specific use cases](https://www.rfc-editor.org/rfc/rfc9562.html#name-uuid-version-8)".  The RFC does not define a creation algorithm for them, which is why this package does not offer a `v8()` method.  The `validate()` and `version()` methods do work with such UUIDs, however.

### uuid.validate(str)

Test a string to see if it is a valid UUID

|           |                                                     |
| --------- | --------------------------------------------------- |
| `str`     | `String` to validate                                |
| _returns_ | `true` if string is a valid UUID, `false` otherwise |

Example:

```javascript
import { validate as uuidValidate } from 'uuid';

uuidValidate('not a UUID'); // ⇨ false
uuidValidate('6ec0bd7f-11c0-43da-975e-2a8ad9ebae0b'); // ⇨ true
```

Using `validate` and `version` together it is possible to do per-version validation, e.g. validate for only v4 UUIds.

```javascript
import { version as uuidVersion } from 'uuid';
import { validate as uuidValidate } from 'uuid';

function uuidValidateV4(uuid) {
  return uuidValidate(uuid) && uuidVersion(uuid) === 4;
}

const v1Uuid = 'd9428888-122b-11e1-b85c-61cd3cbb3210';
const v4Uuid = '109156be-c4fb-41ea-b1b4-efe1671c5836';

uuidValidateV4(v4Uuid); // ⇨ true
uuidValidateV4(v1Uuid); // ⇨ false
```

### uuid.version(str)

Detect RFC version of a UUID

|           |                                          |
| --------- | ---------------------------------------- |
| `str`     | A valid UUID `String`                    |
| _returns_ | `Number` The RFC version of the UUID     |
| _throws_  | `TypeError` if `str` is not a valid UUID |

Example:

```javascript
import { version as uuidVersion } from 'uuid';

uuidVersion('45637ec4-c85f-11ea-87d0-0242ac130003'); // ⇨ 1
uuidVersion('6ec0bd7f-11c0-43da-975e-2a8ad9ebae0b'); // ⇨ 4
```

<!-- prettier-ignore -->
> [!NOTE]
> This method returns `0` for the `NIL` UUID, and `15` for the `MAX` UUID.

## Command Line

UUIDs can be generated from the command line using `uuid`.

```shell
$ npx uuid
ddeb27fb-d9a0-4624-be4d-4615062daed4
```

The default is to generate version 4 UUIDS, however the other versions are supported. Type `uuid --help` for details:

```shell
$ npx uuid --help

Usage:
  uuid
  uuid v1
  uuid v3 <name> <namespace uuid>
  uuid v4
  uuid v5 <name> <namespace uuid>
  uuid v7
  uuid --help

Note: <namespace uuid> may be "URL" or "DNS" to use the corresponding UUIDs
defined by RFC9562
```

## `options` Handling for Timestamp UUIDs

Prior to `uuid@11`, it was possible for `options` state to interfere with the internal state used to ensure uniqueness of timestamp-based UUIDs (the `v1()`, `v6()`, and `v7()` methods). Starting with `uuid@11`, this issue has been addressed by using the presence of the `options` argument as a flag to select between two possible behaviors:

- Without `options`: Internal state is utilized to improve UUID uniqueness.
- With `options`: Internal state is **NOT** used and, instead, appropriate defaults are applied as needed.

## Support

**Browsers**: `uuid` [builds are tested](/uuidjs/uuid/blob/main/wdio.conf.js) against the latest version of desktop Chrome, Safari, Firefox, and Edge. Mobile versions of these same browsers are expected to work but aren't currently tested.

**Node**: `uuid` releases are tested against [Node LTS versions](https://github.com/nodejs/Release), plus one prior.  (E.g. At the time of this writing the oldest LTS version of Node is `node@22`, so `uuid` would support back to `node@20`)

**Typescript**: TS versions released within the past two years are supported. [source](https://github.com/microsoft/TypeScript/issues/49088#issuecomment-2468723715)

## Known issues

<!-- This header is referenced as an anchor in src/rng.ts -->

### "getRandomValues() not supported"

This error occurs in environments where the standard [`crypto.getRandomValues()`](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues) API is not supported. This issue can be resolved by adding an appropriate polyfill:

#### React Native / Expo

1. Install [`react-native-get-random-values`](https://github.com/LinusU/react-native-get-random-values#readme)
1. Import it _before_ `uuid`. Since `uuid` might also appear as a transitive dependency of some other imports it's safest to just import `react-native-get-random-values` as the very first thing in your entry point:

```javascript
import 'react-native-get-random-values';
import { v4 as uuidv4 } from 'uuid';
```

---

Generated from [README_js.md](README_js.md) by [`runmd`](https://github.com/broofa/runmd)
