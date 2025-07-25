
# STX-SkillRep - Decentralized Skill Reputation System – Smart Contract

This Clarity smart contract implements a decentralized reputation and skill assessment platform on the **Stacks blockchain**. Users can register, request assessments for their skills, and get evaluated by peers. Reputation is algorithmically managed based on assessment validity.

---

## 📚 Table of Contents

* [📦 Features](#features)
* [🏗️ Architecture](#architecture)
* [📘 Data Structures](#data-structures)
* [⚙️ Functions](#functions)

  * [Private Functions](#private-functions)
  * [Public Functions](#public-functions)
* [🔐 Access Control](#access-control)
* [📊 Reputation Logic](#reputation-logic)
* [🚧 Validation & Limits](#validation--limits)
* [📁 Example Workflow](#example-workflow)
* [❗ Errors](#errors)
* [🧪 Testing Recommendations](#testing-recommendations)
* [📜 License](#license)

---

## 📦 Features

* 🧑‍💻 **User Registration**
* 🧠 **Skill Management** (by contract owner only)
* 📝 **Skill Assessment Requests**
* 📈 **Peer-Based Scoring with Statistical Analysis**
* ⭐ **Reputation System**
* 🚫 **Invalid Assessment Detection & Penalization**

---

## 🏗️ Architecture

The contract manages the following core concepts:

| Entity              | Description                                     |
| ------------------- | ----------------------------------------------- |
| `users`             | Registered users with reputation data           |
| `skills`            | Skill definitions created by the contract owner |
| `skill-assessments` | Mapping of skill assessment sessions per user   |
| `skill-reputation`  | Skill-specific reputation for each user         |

---

## 📘 Data Structures

```clojure
;; users map
(user principal) => {
    registered: bool,
    skills: (list ...), ;; [not yet used]
    reputation: uint,
    total-assessments: uint,
    invalid-assessments: uint
}

;; skills map
(skill-id uint) => {
    name: string-ascii(50),
    description: string-ascii(200),
    required-assessments: uint,
    category: string-ascii(50)
}

;; skill-assessments map
{skill-id: uint, user: principal} => {
    assessors: list(principal),
    scores: list(uint),
    verified: bool,
    timestamp: uint,
    mean-score: uint,
    standard-deviation: uint
}

;; skill-reputation map
{user: principal, skill-id: uint} => {
    reputation: uint,
    assessments-given: uint,
    valid-assessments: uint
}
```

---

## ⚙️ Functions

### 🔒 Private Functions

#### `update-user-reputation(user, skill-id, is-valid)`

Updates a user's **overall** and **skill-specific** reputation based on whether their score was within a statistical threshold of the mean.

#### `get-next-skill-id()`

Returns the next available skill ID and increments the counter.

#### `process-assessor-reputation(assessor, score, mean, std-dev, skill-id)`

Calculates the deviation of an assessor's score from the mean and updates their reputation accordingly.

---

### 🌐 Public Functions

#### `register-user()`

Registers a new user. Fails if the user is already registered.

#### `add-skill(name, description, required-assessments, category)`

Allows the contract owner to add a new skill for assessment. Performs various input validations.

#### `request-assessment(skill-id)`

Allows a registered user to request a new skill assessment.

#### `submit-assessment(skill-id, user, score)`

Allows a registered user (not the assessee) to submit a score for a user's skill. Performs validation on duplicates, limits, and score bounds. Automatically updates the statistical metrics.

---

## 🔐 Access Control

* `add-skill` is restricted to `contract-owner`.
* All other public functions are permissionless but gated by registration or validation checks.

---

## 📊 Reputation Logic

Reputation is updated using the following logic:

* If the assessor’s score is **within an acceptable deviation** (threshold) of the mean score:

  * ✅ Reward points are added to their reputation.
* If the score deviates **too much from the mean**:

  * ❌ Reputation penalty is applied.

Both **overall** and **skill-specific** reputations are updated:

```clojure
reputation := if (is-valid)
    then (+ reputation reward)
    else (max 0 (- reputation penalty))
```

---

## 🚧 Validation & Limits

* Max assessors per assessment: `20`
* Valid score range: `0 <= score <= 100`
* A user cannot assess themselves.
* Duplicate assessments from the same user are prevented.
* Assessments are only accepted if the skill exists and the user is registered.

---

## 📁 Example Workflow

1. **User A** registers using `register-user`.
2. Contract owner adds "Web Development" using `add-skill`.
3. User A requests assessment via `request-assessment`.
4. Users B, C, and D submit scores for User A using `submit-assessment`.
5. Each assessor's score is checked against the mean:

   * If within acceptable standard deviation → reward
   * Else → penalize

---

## ❗ Errors

Common errors triggered by assertions:

* `err-already-registered`
* `err-not-authorized`
* `err-invalid-score`
* `err-invalid-input`
* `err-invalid-skill-id`
* `err-max-assessors-reached`
* `err-not-registered`
* `err-already-assessed`

---
