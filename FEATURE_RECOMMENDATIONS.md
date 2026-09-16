# Project Q — Feature Implementation Guide
## Curated Open-Source References & Recommendations
**Date:** September 16, 2026  
**Purpose:** Identify proven patterns and ready-to-integrate features from GitHub repositories

---

## Executive Summary

Based on a scan of leading GitHub education platforms aligned with your tech stack (Next.js, NestJS, PostgreSQL, Prisma), we've identified **4 primary feature categories** with proven implementations:

1. **Adaptive Learning & RAG-Grounded Curriculum**
2. **Assessment & Quiz Generation with Analytics**
3. **Parent-Teacher Communication & Notifications**
4. **Privacy Controls & Guardian Consent** (COPPA/GDPR-K compliant)

Each recommendation includes a specific GitHub repository, the features worth adopting, and integration guidance for Project Q.

---

## 1. ADAPTIVE LEARNING & CURRICULUM-GROUNDED TUTORING

### Primary Recommendation: AI Learning Path Generator
**Repository:** [arun3676/ai-learning-path-generator](https://github.com/arun3676/ai-learning-path-generator)

**Why it's relevant:**
- Combines **RAG (Retrieval-Augmented Generation)** with adaptive learning path generation
- Personalizes learning paths based on:
  - User-selected topics
  - Expertise level
  - Learning style (visual, auditory, reading/writing, kinesthetic)
  - Available study time
  - Specific learning goals
- Includes **progress tracking** (aligns with Project Q V1.0 requirement)

**Features to adopt:**
- **Adaptive difficulty adjustment** based on learner performance
- **Learning style detection** to personalize Q recommendations
- **Progress heatmaps** for learner and parent dashboards
- **RAG context boundaries** to prevent hallucinations (aligns with Project Q safety goals)

**Integration Pattern for Project Q:**
```
Student Profile → Learning Goals → Diagnostic Quiz → 
  ↓
AI Path Generator (RAG) → Recommended Learning Sequence → Q Tutor Alignment → 
  ↓
Progress Tracking → Parent Notifications → Mastery Evidence
```

**Estimated Implementation Effort:** 3–4 sprints (with curriculum grounding layer)

---

### Secondary Recommendation: OpenRAG Framework
**Repository:** [OpenRAG Research](https://ieeexplore.ieee.org/document/10900069)  
**Reference:** [Awesome RAG Curated List](https://github.com/Danielskry/Awesome-RAG)

**Why relevant:**
- Modular open-source RAG system specifically designed for personalized learning
- Components include:
  - **Generator** (LLM interface for Q)
  - **Indexing subsystem** (curriculum document grounding)
  - **Retriever** (fetch relevant learning materials)
  - **Orchestration module** (adapt based on learner progress)

**Features to adopt:**
- **Curriculum grounding layer** — ensure Q only references approved materials
- **Source attribution** — show learners where Q's answers come from
- **Retrieval confidence scoring** — suppress low-confidence answers
- **Fallback to teacher-curated content** when AI confidence is low

**Integration with Project Q Q Component:**
- Replace generic LLM with curriculum-grounded RAG layer
- Add metadata tracking (source, confidence, curriculum alignment)
- Route uncertain queries to teacher for manual response

---

## 2. ASSESSMENT & QUIZ GENERATION WITH LEARNING ANALYTICS

### Primary Recommendation: E-learning Platform (Full Stack Reference)
**Repository:** [f-a-t-h-e/E-learning-platform](https://github.com/f-a-t-h-e/E-learning-platform)

**Why it's relevant:**
- Full NestJS + PostgreSQL + Prisma backend (your stack)
- Implements:
  - Quiz creation and management
  - Automatic and manual grading
  - Question banks with difficulty tagging
  - Real-time analytics dashboard
  - Student progress tracking
  - Reporting for teachers and parents

**Features to adopt:**
```typescript
// Quiz Schema Pattern (from E-learning Platform)
- Quiz (id, title, subject, difficulty_level, time_limit)
- Question (id, quiz_id, type, content, options, correct_answer, difficulty)
- Submission (id, student_id, quiz_id, score, time_spent, date)
- QuizAnalytics (question_id, avg_score, discrimination, miss_rate)
```

**Teacher-Facing Analytics:**
- Question difficulty and discrimination index
- Most-missed questions by topic
- Class average vs. individual performance
- Time-on-task analysis

**Parent-Facing Analytics (V1):**
- Quiz performance over time (line chart)
- Topic mastery breakdown (skill badges)
- Comparative performance (class percentile)
- Mastery status (not yet / approaching / mastered)

**Estimated Implementation Effort:** 2–3 sprints

---

### Secondary: Automated Quiz Generation
**Related Pattern:** Question generation from content  
**Reference:** [pypast/question_generation](https://github.com/pypast/question_generation)

**Implementation ideas:**
```
Learning Material (PDF/Video) 
  → NLP Pipeline (via question_generation repo) 
  → Generated Question Bank 
  → Teacher Review & Approval 
  → Student Attempts
```

**Recommended Features:**
- Multiple-choice, true/false, short-answer question types
- Automatic randomization of answer options
- Scaffolded difficulty (hint system tied to Q)
- Adaptive next-question selection

---

## 3. PARENT-TEACHER COMMUNICATION & NOTIFICATIONS

### Primary Recommendation: OpenSIS (Student Information System)
**Repository:** [OS4ED/openSIS-Classic](https://github.com/OS4ED/openSIS-Classic)

**Why relevant:**
- Dedicated parent, teacher, and student portals
- Built-in **message center** for direct communication
- **Automatic notifications** for:
  - Assignment submissions
  - Grade releases
  - Attendance changes
  - Announcements
  - Meetings

**Features to adopt:**
```
Assignment Created → Student Notified
  ↓
Student Submits → Teacher Notified
  ↓
Teacher Grades → Parent Notified (if enabled)
  ↓
Parent Views Grade → Automatic Engagement Log
```

**For Project Q V1.0:**
- **Learner Notifications:**
  - Quiz available
  - New assignment
  - Q recommendation
  - Achievement unlocked
  - Progress milestone

- **Parent Notifications:**
  - Child completed quiz/assignment
  - Academic milestone reached
  - Teacher message
  - Attendance/participation summary (weekly digest)

- **Teacher Notifications:**
  - Student submission received
  - Class announcement time
  - Parent requested meeting

**Integration Pattern:**
```typescript
// Notification Hub Architecture
Event Triggered (quiz complete) 
  → Notification Service 
  → Route to Target Role (student, parent, teacher)
  → Apply Privacy Filtering (show only authorized data)
  → Send via Preference Channel (email, in-app, SMS if enabled)
```

**Estimated Implementation Effort:** 2–3 sprints

---

### Secondary: Moodle Messaging & Plugins
**Repository:** [moodle/moodle](https://github.com/moodle/moodle)

**Why reference:**
- Mature messaging system with role-based filtering
- Extensive notification system (customizable per role)
- Parent role plugin available
- Research-backed access control patterns

---

## 4. PRIVACY CONTROLS & GUARDIAN CONSENT (COPPA/GDPR-K Compliance)

### Implementation Pattern: Age-Gating + Verifiable Consent
**Reference Base:** GitHub community TypeScript/React patterns

**Critical for Project Q V1.0 Launch:**

#### Phase 1: Age Detection & Consent Gating

```typescript
// Component: AgeGateFlow.tsx
// 1. User inputs DOB
// 2. If under 13 (COPPA) or under 16 (GDPR-K):
//    → Show Guardian Consent Form
// 3. Send verification email to guardian
// 4. Upon verification: 
//    → Activate student account
//    → Apply age-appropriate restrictions

interface ConsentRecord {
  child_id: UUID;
  guardian_email: string;
  verified_at: Timestamp;
  consent_version: string;
  ip_address: string;
  verification_token: UUID;
  consent_types: ConsentType[];
}

enum ConsentType {
  DATA_COLLECTION = "data_collection",
  MARKETING = "marketing",
  THIRD_PARTY_SHARING = "third_party_sharing",
}
```

#### Phase 2: Guardian Privacy Dashboard

```typescript
interface GuardianPrivacyControls {
  // View
  viewChildData: () => Promise<StudentDataExport>;
  viewActivityLog: () => Promise<ActivityLog[]>;
  viewAIConversations: () => Promise<ConversationSummary[]>;
  
  // Control
  restrictDataCollection: (dataTypes: string[]) => Promise<void>;
  deleteChildData: (dataTypes: string[]) => Promise<void>;
  revokeConsent: () => Promise<void>;
  
  // Permissions
  allowThirdPartySharing: boolean; // default: false
  shareProgressWithSchool: boolean; // default: true
  shareProgressWithTeacher: boolean; // default: true
}
```

**Key Design Principles:**
- **Default to private:** Teachers/parents must explicitly enable data sharing
- **Granular controls:** Separate consent for tutoring, messaging, data retention
- **Easy revocation:** One-click consent withdrawal
- **Audit trail:** Every privacy action logged with timestamp and IP
- **Transparent storage:** Show guardians exactly what data is stored and for how long

#### Phase 3: Data Deletion & Retention Policies

```typescript
enum RetentionPolicy {
  ACTIVE_LEARNING = "active", // While enrolled/learning (never auto-delete)
  ARCHIVE_1_YEAR = "archive_1yr", // 1 year after completion
  ARCHIVE_7_YEAR = "archive_7yr", // Compliance requirement
  IMMEDIATE_DELETE = "immediate", // Upon guardian request
}

interface DataLifecycleEvent {
  user_id: UUID;
  event_type: "collection" | "use" | "retention" | "deletion";
  data_category: string; // quiz_response, ai_conversation, attendance, etc.
  timestamp: Timestamp;
  retention_policy: RetentionPolicy;
  deletion_date?: Timestamp;
}
```

**Estimated Implementation Effort:** 3–4 sprints (includes legal/compliance review)

---

## 5. REFERENCE ARCHITECTURES: TECH STACK ALIGNMENT

### Option A: Follow E-learning Platform Stack Exactly
**Repository:** [f-a-t-h-e/E-learning-platform](https://github.com/f-a-t-h-e/E-learning-platform)

**Stack Alignment:**
- ✅ NestJS backend (your choice)
- ✅ PostgreSQL + Prisma (your choice)
- ✅ React frontend (your choice)
- ✅ Real-time capabilities (Socket.io for notifications)

**Why this option:**
- Minimal translation effort
- Proven error patterns and solutions
- Database schemas ready to adapt
- Authentication/authorization patterns match your needs

---

### Option B: Build on Vercel's Next.js + Prisma Starter
**Repository:** [vercel/nextjs-postgres-auth-starter](https://github.com/vercel/nextjs-postgres-auth-starter)

**Stack Alignment:**
- ✅ Next.js + Prisma (your choice)
- ✅ PostgreSQL (your choice)
- ✅ Authentication patterns (Neon + Google/Auth0 support)
- ✅ Vercel deployment (your hosting)

**Why this option:**
- Cleanest starter for your frontend + backend integration
- Minimal boilerplate
- Built for Vercel/Neon ecosystem

---

## 6. FEATURE ROADMAP: 30-90 DAY IMPLEMENTATION PLAN

### Sprint 1–2 (Sep 21 – Oct 18)
**Focus:** Security + Operational Foundation

- ✅ (Already planned) Runtime input validation
- ✅ (Already planned) AI prompt-injection hardening
- **NEW:** Add privacy consent logging layer (foundation)

---

### Sprint 3–4 (Oct 19 – Nov 15)
**Focus:** Teacher & Parent Core Journeys

**New Recommendations:**
- Teacher quiz creation tool (from E-learning Platform reference)
- Assignment submission + grading flow
- Parent notification preferences
- Guardian privacy dashboard (Phase 1: view data)

---

### Sprint 5–6 (Nov 16 – Dec 13)
**Focus:** Analytics & Adaptive Learning

**New Recommendations:**
- Learning analytics dashboard (teacher-facing)
- Quiz analytics (question difficulty, discrimination)
- Learner progress visualization
- **PILOT:** AI Learning Path Generator integration (from recommendation #1)

---

### Sprint 7–8 (Dec 14 – Jan 10)
**Focus:** Privacy & Legal Compliance

**New Recommendations:**
- Complete Guardian Consent flows (Phases 1–2)
- Data deletion/export APIs
- Retention policy enforcement
- Legal/compliance documentation

---

### Sprint 9–10 (Jan 11 – Feb 7)
**Focus:** Independent Learner & Full System Regression

**New Recommendations:**
- Goal-based onboarding (AI Path Generator for independent learners)
- Mastery tracking against learning standards
- Learner portfolio basics
- Full privacy regression testing

---

## 7. QUICK-START: HOW TO INTEGRATE EACH FEATURE

### For AI Learning Path Adapter (30-minute integration)

```bash
# 1. Study the AI Learning Path Generator
git clone https://github.com/arun3676/ai-learning-path-generator.git

# 2. Extract the path generation algorithm
# Location: Likely in src/services/pathGenerator.ts

# 3. Adapt for your data model:
# Your Learner → Their Goals → Their Level → Our Path Generator → Q Alignment

# 4. Add to your NestJS API:
# POST /api/v1/learner/:id/generate-path
# Input: { goal, level, learningStyle, availableHours }
# Output: [ { topicId, estimatedHours, difficulty, resourceIds } ]
```

---

### For Quiz Analytics Dashboard (2-sprint integration)

```bash
# 1. Fork the E-learning Platform
git clone https://github.com/f-a-t-h-e/E-learning-platform.git

# 2. Extract quiz and analytics schemas
# Location: Likely in src/modules/quiz

# 3. Adapt Prisma schema to Project Q data model
# Example: Map their QuizAnalytics to your Progress/Mastery tracking

# 4. Implement teacher dashboard using same UI patterns
# Reuse: Question analysis, class average charts, student performance matrix
```

---

### For Parent Notifications (1-sprint integration)

```bash
# 1. Adopt OpenSIS notification patterns
# Core: Event → Filter by Role → Apply Privacy Rules → Send

# 2. Implement in Project Q:
# POST /api/v1/notifications
# {
#   "event": "quiz_completed",
#   "student_id": "xxx",
#   "target_roles": ["parent", "teacher"],
#   "privacy_level": "summary" // Don't show raw answers
# }

# 3. Create notification preferences UI for parents
# Allow: frequency (realtime/daily/weekly), channels (email/in-app)
```

---

### For Guardian Consent (2-sprint integration)

```bash
# 1. Create Prisma schema for consent records:
model ConsentRecord {
  id String @id @default(cuid())
  childId String @db.Uuid
  guardianEmail String
  verifiedAt DateTime?
  consentVersion String
  consentTypes ConsentType[]
  ipAddress String
  createdAt DateTime @default(now())
}

# 2. Implement verification flow:
# /auth/signup (minor) → /auth/guardian-consent → Email Verification → Account Active

# 3. Add to guardian dashboard:
# /guardian/privacy → View Data / Delete Data / Revoke Consent
```

---

## 8. RISK MITIGATION & COMPLIANCE CHECKLIST

### Before V1.0 Launch:

- [ ] **Privacy Legal Review:** Have external privacy counsel review consent forms
- [ ] **COPPA Compliance:** Age-gating, verifiable parental consent, data minimization
- [ ] **GDPR-K (UK ICO):** Children's code compliance review
- [ ] **FERPA:** School data handling review (with legal team)
- [ ] **AI Provider Terms:** Confirm data retention policies for LLM providers (OpenAI, Anthropic, etc.)
- [ ] **Accessibility (WCAG 2.1 AA):** Quiz interface, analytics dashboards, consent forms
- [ ] **Data Security:** Encryption, secure deletion, incident response plan
- [ ] **Pilot Validation:** Run features with 50–100 real users before full launch

---

## 9. SUMMARY: PRIORITY RANKING

| Feature | Effort | Impact | V1.0 Must-Have | Recommended Repo |
|---------|--------|--------|----------------|------------------|
| Adaptive Learning Paths | 3–4 sprints | High | Yes | arun3676/ai-learning-path-generator |
| Quiz Analytics Dashboard | 2–3 sprints | High | Yes | f-a-t-h-e/E-learning-platform |
| Parent Notifications | 2–3 sprints | High | Yes | OS4ED/openSIS-Classic |
| Guardian Consent & Privacy | 3–4 sprints | Critical | Yes | Community patterns (TypeScript/React) |
| RAG Curriculum Grounding | 2–3 sprints | High | Yes | Danielskry/Awesome-RAG |
| Teacher Quiz Builder | 1–2 sprints | Medium | Yes | f-a-t-h-e/E-learning-platform |
| Learning Passport | 1–2 sprints | Medium | V0.9 | Custom + learner portfolio patterns |
| Independent Learner Mode | 2–3 sprints | Medium | Yes | arun3676/ai-learning-path-generator |

---

## 10. NEXT STEPS

1. **Review the recommended repositories** (30 min each):
   - [ ] AI Learning Path Generator
   - [ ] E-learning Platform
   - [ ] OpenSIS
   - [ ] Awesome RAG

2. **Create GitHub Issues for each feature** with:
   - Link to reference repository
   - Feature acceptance criteria
   - Sprint assignment
   - Integration points

3. **Schedule integration planning session** with your team to:
   - Validate data model alignment
   - Identify schema conflicts
   - Assign feature ownership

4. **Document any deviations** from reference implementations in your project's ADR (Architecture Decision Record) folder.

---

**Prepared by:** GitHub Copilot  
**For:** Project Q Development Team  
**Status:** Ready for sprint planning integration

