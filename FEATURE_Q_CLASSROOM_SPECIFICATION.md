# Project Q — Online Classroom Module (Q Classroom)
## Live Teaching Platform Specification & Implementation Guide

**Feature Name:** Q Classroom (Live Online Teaching)  
**Status:** NEW FEATURE PROPOSAL  
**Target Version:** V1.1 (Post-Launch) OR Pilot Feature V0.95  
**Priority:** High  
**Estimated Effort:** 10–14 sprints (5–7 months)

---

## 1. Executive Summary

**Q Classroom** is a built-in, school-hosted live online teaching platform that enables:
- **Teachers** to conduct real-time video classes with students
- **Students** to attend live classes with their peers
- **Interactive features:** Screen sharing, whiteboard, polls, breakout rooms, attendance tracking
- **Recording & Playback:** Auto-record classes for asynchronous learners
- **Integration with Q Learning:** Direct linkage between live classes and assignments, quizzes, progress tracking

### Why This Matters for Project Q
- **Complete Learning Ecosystem:** Live teaching + AI tutoring + assignments + assessment = unified platform
- **Attendance Tracking:** Automatic evidence of learner engagement
- **Hybrid Learning:** Support both synchronous (live) and asynchronous (recorded) learning
- **Privacy-First:** School-hosted, no data sold to third parties (vs. Zoom, Google Meet)
- **Cost Savings:** No per-student licensing fees
- **Teacher Empowerment:** Full control over class recordings, student data

---

## 2. Product Vision: Q Classroom Within Project Q

### Differentiation from Commercial Platforms

| Aspect | Zoom/Google Meet | Q Classroom |
|--------|------------------|-------------|
| **Hosting** | Cloud (vendor-controlled) | School-hosted or VIEQAN-managed |
| **Student Data** | Used for analytics/ads | School-owned, never monetized |
| **Integration** | Calendar API only | Native LMS integration (assignments, grades, progress) |
| **Recording** | Cloud storage | School-owned, can be auto-published to Q Social |
| **Cost Model** | Per-user licensing | Included in school subscription |
| **Attendance** | Manual tracking | Automatic + linked to progress |
| **Breakout Rooms** | Premium feature | Included standard |
| **Whiteboard** | Basic | Rich (built-in + integrations with Excalidraw) |
| **Teacher Controls** | Limited | Full: admit/remove, mute all, lock class, etc. |

---

## 3. Core Features: MVP (V0.95 Pilot)

### 3.1 Classroom Session Management

```typescript
interface ClassroomSession {
  id: UUID;
  classId: UUID; // Linked to Q Class
  class: Class;
  
  title: string;
  description?: string;
  
  // Schedule
  scheduledStartTime: DateTime;
  scheduledEndTime: DateTime;
  actualStartTime?: DateTime;
  actualEndTime?: DateTime;
  
  // Room Configuration
  hostId: UUID; // Teacher
  host: User;
  
  roomCode: string; // 6-char code (e.g., ABC123)
  sessionLink: string; // https://q.school.edu/classroom/ABC123
  
  // Session State
  status: "scheduled" | "in_progress" | "ended" | "cancelled";
  maxParticipants: number; // Default: class size + buffer
  
  // Features Enabled
  enableVideo: boolean;
  enableAudio: boolean;
  enableScreenShare: boolean;
  enableWhiteboard: boolean;
  enablePolls: boolean;
  enableChat: boolean;
  enableBreakoutRooms: boolean;
  enableRecording: boolean;
  
  // Recording & Playback
  recordingEnabled: boolean;
  recordingStatus?: "not_started" | "recording" | "processing" | "ready";
  recordingUrl?: string;
  recordingStorageId?: string; // S3/GCS reference
  autoPublishRecording: boolean; // Auto-publish to Q Social after class
  
  // Attendance
  expectedParticipants: string[]; // Array of user IDs (enrolled students)
  actualParticipants: Participant[];
  attendanceRecording: AttendanceLog[];
  
  // Chat & Interactions
  isChatPublic: boolean; // Teachers can make chat private/public
  isPollsEnabled: boolean;
  isRaiseHandEnabled: boolean;
  
  // Moderation
  lockedAfterStart: boolean; // No new joins after class starts
  requireApprovalToJoin: boolean;
  mutedByDefault: boolean;
  cameraOffByDefault: boolean;
  
  // Meeting Context
  associatedAssignments?: UUID[]; // Assignments due before/after class
  associatedResources?: string[]; // URLs to materials
  
  created_at: DateTime;
  updated_at: DateTime;
  deleted_at?: DateTime;
}

interface Participant {
  userId: UUID;
  user: User;
  joinedAt: DateTime;
  leftAt?: DateTime;
  duration: number; // in seconds
  
  // Permissions
  isHost: boolean;
  isCoHost?: boolean;
  isMuted: boolean;
  cameraOn: boolean;
  
  // Engagement
  raisedHandAt?: DateTime;
  hasSharedScreen: boolean;
  pollsAnswered: number;
  messagesCount: number;
  
  // Compliance
  recordingConsent: boolean; // Parent/student approved recording
  dataCollectionConsent: boolean; // Approved attendance tracking
}

interface AttendanceLog {
  id: UUID;
  sessionId: UUID;
  userId: UUID;
  joinedAt: DateTime;
  leftAt?: DateTime;
  durationSeconds: number;
  status: "present" | "late" | "absent" | "excused";
  
  // Linked to Progress
  linkedToProgressRecord: boolean;
  progressRecordId?: UUID;
  createdAt: DateTime;
}
```

### 3.2 Classroom Interface Components

```typescript
// Teacher View (Instructor Dashboard)
<ClassroomTeacherView>
  <Header>
    <RoomCode>{roomCode}</RoomCode>
    <RecordingIndicator isRecording={true} />
    <Timer startTime={actualStartTime} />
    <EndSessionButton />
  </Header>
  
  <MainArea>
    <VideoGrid>
      {/* Teacher's video prominently shown, grid of student videos */}
      <TeacherVideoFeed />
      {participants.map(p => (
        <StudentVideoTile 
          key={p.userId}
          participant={p}
          onMute={() => muteParticipant(p.userId)}
          onRemove={() => removeParticipant(p.userId)}
        />
      ))}
    </VideoGrid>
  </MainArea>
  
  <Toolbar>
    <MicrophoneToggle />
    <CameraToggle />
    <ScreenShareButton onClick={startScreenShare} />
    <WhiteboardButton onClick={openWhiteboard} />
    <PollButton onClick={createPoll} />
    <ChatButton />
    <ParticipantsPanel>
      {participants.map(p => (
        <ParticipantRow
          participant={p}
          actions={[muteAll, removeAll, admitWaitlist]}
        />
      ))}
    </ParticipantsPanel>
    <HandRaisedNotifier raisedHands={raisedHands} />
  </Toolbar>
  
  <RightPanel>
    <Tabs>
      <ChatTab messages={messages} onSendMessage={sendMessage} />
      <ParticipantsTab participants={participants} />
      <PollTab polls={polls} onCreatePoll={createPoll} />
    </Tabs>
  </RightPanel>
</ClassroomTeacherView>

// Student View (Attendee)
<ClassroomStudentView>
  <Header>
    <RoomCode>{roomCode}</RoomCode>
    <RecordingIndicator isRecording={true} message="Class is being recorded" />
    <TeacherNameDisplay />
    <LeaveSessionButton />
  </Header>
  
  <MainArea>
    <VideoGrid>
      {/* Teacher's video prominently shown */}
      <TeacherVideoFeed />
      {/* Limited peer videos (e.g., 6-person grid) */}
      {visibleParticipants.map(p => (
        <StudentVideoTile key={p.userId} participant={p} />
      ))}
    </VideoGrid>
  </MainArea>
  
  <Toolbar>
    <MicrophoneToggle />
    <CameraToggle />
    <ScreenShareButton if={allowStudentScreenShare} />
    <RaiseHandButton onClick={raiseHand} raised={handRaised} />
    <ChatButton />
    <MoreOptions>
      <Feedback />
      <Report />
      <LeaveClass />
    </MoreOptions>
  </Toolbar>
  
  <RightPanel>
    <Tabs>
      <ChatTab messages={messages} onSendMessage={sendMessage} />
      <ParticipantsTab count={participantCount} />
      <PollTab polls={polls} onAnswerPoll={answerPoll} />
    </Tabs>
  </RightPanel>
</ClassroomStudentView>
```

### 3.3 Feature Details

#### Screen Sharing
```
Teacher initiates screen share
  ↓
Video feed is replaced with teacher's screen
  ↓
Teacher can still be visible in small PIP (Picture-in-Picture)
  ↓
Students see annotations (optional whiteboard overlay)
  ↓
Recording captures screen + audio
```

#### Whiteboard
```
Teacher clicks "Whiteboard" button
  ↓
New canvas opens (integrated or via Excalidraw iframe)
  ↓
Teacher can draw, write, insert shapes, text
  ↓
Students see in real-time
  ↓
Whiteboard is recorded + can be exported as image
```

#### Polls (Real-Time Engagement)
```
Teacher creates poll mid-class:
  ├── Question: "What is 2+2?"
  ├── Options: ["3", "4", "5", "6"]
  ├── Timer: 30 seconds
  └── Results visibility: Show after all answer OR show live
  
Students see poll notification
  ↓
Students vote
  ↓
Teacher sees results real-time
  ↓
Results saved to attendance/engagement record
  ↓
Can be linked to progress tracking
```

#### Breakout Rooms
```
Teacher creates breakout rooms:
  ├── Room 1: Group A (5 students)
  ├── Room 2: Group B (5 students)
  ├── Room 3: Teacher + Mixed (5 students)
  
Students auto-assigned or teacher manually assigns
  ↓
Each room has its own video/audio/chat/whiteboard
  ↓
Teacher can hop between rooms
  ↓
Timer shows when breakout session ends
  ↓
All students return to main room automatically
  ↓
Attendance & chat logged separately per room
```

#### Chat
```
Public Chat (visible to all):
├── Student messages
├── Teacher messages (can be pinned)
├── System notifications ("John joined")
└── Moderation (teacher can delete, mute user)

Private Chat (1-on-1):
├── Student to Teacher
├── Student to Student (if enabled)
└── Logged in attendance record
```

---

## 4. Integration with Project Q Ecosystem

### 4.1 Class Linkage

```typescript
interface Class {
  // ... existing Q fields
  
  // Classroom integration
  classroomEnabled: boolean;
  sessions: ClassroomSession[]; // All past & scheduled sessions
  
  // Linked to other Q components
  linkedAssignments: Assignment[];
  linkedQuizzes: Quiz[];
  linkedProgressTracking: ProgressRecord[];
  
  // Auto-record settings
  autoRecordClasses: boolean;
  autoPublishRecordingsToFeed: boolean; // Auto-post to Q Social
  recordingRetentionDays: number; // Default: 180
}
```

### 4.2 Attendance Sync

```
Classroom Session Ends
  ↓
Attendance Log Created:
├── Student A: Present (45 min)
├── Student B: Present (42 min, 3 min late)
├── Student C: Absent
└── Student D: Present (early leave 30 min)
  ↓
Auto-update Progress Record:
├── Engagement: +45 min
├── Participation Score: +points for attendance
├── Attendance Streak: +1 day
  ↓
Notify:
├── Parent: "Your child attended Math class (45 min)"
├── Teacher: Attendance summary
└── Student: Attendance badge
```

### 4.3 Recording → Q Social Integration

```
Teacher ends class recording
  ↓
Video processing (transcoding to streaming format)
  ↓
Auto-publish to Q Social (if enabled):
├── Title: "[Class Recording] {class_name} - {date}"
├── Visibility: Set by teacher (private/class/school)
├── Curriculum tags: Auto-set from class metadata
├── Description: "Class materials: [links to assignments/resources]"
└── Thumbnail: Auto-generated from first frame
  ↓
Students & parents can view recorded classes
  ↓
Teacher gets engagement metrics (views, downloads)
```

---

## 5. Data Model (Prisma Schema)

```prisma
model ClassroomSession {
  id            String    @id @default(cuid())
  classId       String    @db.Uuid
  class         Class     @relation(fields: [classId], references: [id])
  
  title         String
  description   String?
  
  scheduledStartTime DateTime
  scheduledEndTime   DateTime
  actualStartTime    DateTime?
  actualEndTime      DateTime?
  
  hostId        String    @db.Uuid
  host          User      @relation("ClassroomHostSessions", fields: [hostId], references: [id])
  
  roomCode      String    @unique
  sessionLink   String
  
  status        ClassroomStatus @default(SCHEDULED)
  maxParticipants Int
  
  // Feature Flags
  enableVideo           Boolean @default(true)
  enableAudio           Boolean @default(true)
  enableScreenShare     Boolean @default(true)
  enableWhiteboard      Boolean @default(true)
  enablePolls           Boolean @default(true)
  enableChat            Boolean @default(true)
  enableBreakoutRooms   Boolean @default(true)
  enableRecording       Boolean @default(true)
  
  // Recording
  recordingEnabled      Boolean @default(true)
  recordingStatus       RecordingStatus?
  recordingUrl          String?
  recordingStorageId    String?
  autoPublishRecording  Boolean @default(false)
  recordingRetentionDays Int    @default(180)
  
  // Session Configuration
  lockAfterStart        Boolean @default(false)
  requireApprovalToJoin Boolean @default(false)
  mutedByDefault        Boolean @default(false)
  cameraOffByDefault    Boolean @default(false)
  
  isChatPublic          Boolean @default(true)
  isPollsEnabled        Boolean @default(true)
  isRaiseHandEnabled    Boolean @default(true)
  
  // Associated Content
  associatedAssignments String[] @db.Uuid
  associatedResources   String[]
  
  // Relationships
  participants          Participant[]
  attendanceLogs        AttendanceLog[]
  chatMessages          ChatMessage[]
  polls                 Poll[]
  breakoutRooms         BreakoutRoom[]
  
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt
  deletedAt             DateTime?
  
  @@index([classId])
  @@index([hostId])
  @@index([status])
  @@index([roomCode])
}

model Participant {
  id                String    @id @default(cuid())
  sessionId         String
  session           ClassroomSession @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  
  userId            String    @db.Uuid
  user              User      @relation("ClassroomParticipants", fields: [userId], references: [id])
  
  joinedAt          DateTime
  leftAt            DateTime?
  durationSeconds   Int
  
  isHost            Boolean   @default(false)
  isCoHost          Boolean   @default(false)
  isMuted           Boolean   @default(false)
  cameraOn          Boolean   @default(false)
  
  raisedHandAt      DateTime?
  hasSharedScreen   Boolean   @default(false)
  pollsAnswered     Int       @default(0)
  messagesCount     Int       @default(0)
  
  recordingConsent  Boolean   @default(true)
  dataCollectionConsent Boolean @default(true)
  
  createdAt         DateTime  @default(now())
  
  @@unique([sessionId, userId])
  @@index([sessionId])
  @@index([userId])
}

model AttendanceLog {
  id                String    @id @default(cuid())
  sessionId         String
  session           ClassroomSession @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  
  userId            String    @db.Uuid
  user              User      @relation("AttendanceLogs", fields: [userId], references: [id])
  
  joinedAt          DateTime
  leftAt            DateTime?
  durationSeconds   Int
  
  status            AttendanceStatus @default(PRESENT)
  linkedToProgressRecord Boolean @default(false)
  progressRecordId  String?   @db.Uuid
  
  createdAt         DateTime  @default(now())
  
  @@index([sessionId])
  @@index([userId])
  @@index([status])
}

model ChatMessage {
  id            String    @id @default(cuid())
  sessionId     String
  session       ClassroomSession @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  
  authorId      String    @db.Uuid
  author        User      @relation("ClassroomChatMessages", fields: [authorId], references: [id])
  
  content       String    @db.Text
  messageType   ChatMessageType @default(PUBLIC)
  recipientId   String?   @db.Uuid // For private messages
  
  isPinned      Boolean   @default(false)
  isHidden      Boolean   @default(false)
  
  createdAt     DateTime  @default(now())
  
  @@index([sessionId])
  @@index([authorId])
}

model Poll {
  id            String    @id @default(cuid())
  sessionId     String
  session       ClassroomSession @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  
  createdBy     String    @db.Uuid
  creator       User      @relation("CreatedPolls", fields: [createdBy], references: [id])
  
  question      String
  options       String[]
  responses     PollResponse[]
  
  timerSeconds  Int       @default(30)
  showResultsLive Boolean  @default(false)
  
  createdAt     DateTime  @default(now())
  closedAt      DateTime?
  
  @@index([sessionId])
}

model PollResponse {
  id        String    @id @default(cuid())
  pollId    String
  poll      Poll      @relation(fields: [pollId], references: [id], onDelete: Cascade)
  
  userId    String    @db.Uuid
  user      User      @relation("PollResponses", fields: [userId], references: [id])
  
  selectedOption String
  
  createdAt DateTime  @default(now())
  
  @@unique([pollId, userId])
}

model BreakoutRoom {
  id            String    @id @default(cuid())
  sessionId     String
  session       ClassroomSession @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  
  name          String
  order         Int
  
  participants  String[]  @db.Uuid // Array of user IDs
  
  startedAt     DateTime?
  endedAt       DateTime?
  
  createdAt     DateTime  @default(now())
  
  @@index([sessionId])
}

enum ClassroomStatus {
  SCHEDULED
  IN_PROGRESS
  ENDED
  CANCELLED
}

enum RecordingStatus {
  NOT_STARTED
  RECORDING
  PROCESSING
  READY
  FAILED
}

enum AttendanceStatus {
  PRESENT
  LATE
  ABSENT
  EXCUSED
}

enum ChatMessageType {
  PUBLIC
  PRIVATE
  SYSTEM
}
```

---

## 6. Technical Architecture

### 6.1 WebRTC & Signaling Architecture

```
Classroom Server (NestJS)
│
├── Signaling Service (WebSocket)
│   ├── Handle participant join/leave
│   ├── Distribute SDP offers/answers
│   ├── Exchange ICE candidates
│   └── Relay chat/polls
│
├── Recording Service (Optional external)
│   ├── Capture video streams
│   ├── Encode/transcode
│   └── Store to S3/GCS
│
├── Media Server (TURN server)
│   ├── ICE candidate collection
│   ├── NAT traversal
│   └── Relay if P2P fails
│
└── Database (PostgreSQL)
    ├── Session metadata
    ├── Attendance logs
    └── Chat history
```

### 6.2 Frontend Architecture (Next.js/React)

```
Components:
├── ClassroomTeacherView
│   ├── VideoGrid (teacher's video + student videos)
│   ├── Toolbar (mute, camera, screen share, whiteboard)
│   ├── ParticipantsPanel (list with manage options)
│   └── ChatPanel (public + private messages)
│
├── ClassroomStudentView
│   ├── VideoGrid (teacher + visible peers)
│   ├── Toolbar (mute, camera, raise hand)
│   ├── ChatPanel (view messages only)
│   └── PollWidget (answer polls)
│
├── ClassroomSetup (scheduling, settings)
├── ClassroomHistory (past sessions, recordings)
└── AttendanceReport (for teachers)

Libraries:
├── simple-peer (WebRTC P2P)
├── Socket.io (signaling, real-time)
├── React Hooks (state management)
└── Tailwind CSS (styling)
```

---

## 7. Open-Source Technology Recommendations

### 7.1 Primary Reference: BigBlueButton

**Repository:** [bigbluebutton/bigbluebutton](https://github.com/bigbluebutton/bigbluebutton)  
**Why Use:**
- Purpose-built for education
- Includes ALL features: whiteboard, breakout rooms, polling, screen share
- LMS integrations (Moodle, Canvas, etc.)
- Recording + playback
- Scalable architecture
- Active community & regular updates

**What to Extract:**
- Session management patterns
- Whiteboard implementation
- Recording architecture
- Breakout room logic
- Poll system

**When to Use:** If building from scratch, can run BigBlueButton server and build lightweight Next.js UI wrapper

---

### 7.2 Alternative: Jitsi Meet (Lighter Alternative)

**Repository:** [jitsi/jitsi-meet](https://github.com/jitsi/jitsi-meet)  
**Why Use:**
- Simpler architecture than BigBlueButton
- Easy to customize UI (React-based)
- Good for small-medium classrooms (<30 people)
- Easier to deploy

**What to Extract:**
- Video conferencing patterns
- UI component library
- Screen share implementation

**Limitation:** No built-in whiteboard (must integrate separately, e.g., Excalidraw)

---

### 7.3 WebRTC-Based: Samvaad (Custom Building)

**Repository:** [Hashim-stack/samvaad-webrtc-classroom](https://github.com/Hashim-stack/samvaad-webrtc-classroom)  
**Why Use:**
- Uses Next.js (your stack)
- WebRTC + Socket.io (your preference)
- Modular components
- Good starting point for custom implementation

**What to Extract:**
- WebRTC signaling patterns
- Socket.io event flow
- React component structure
- Peer connection management

**Limitation:** Missing some features (recording, breakout rooms) — need to extend

---

### 7.4 Whiteboard Integration: Excalidraw

**Repository:** [excalidraw/excalidraw](https://github.com/excalidraw/excalidraw)  
**Why Use:**
- Excellent collaborative whiteboard
- Can be embedded as iframe or component
- Real-time collaboration via WebSocket
- Works standalone or integrated

**Implementation Pattern:**
```typescript
// Embed in ClassroomSession
<Whiteboard>
  <ExcalidrawComponent
    onChange={handleWhiteboardUpdate}
    onPointerUpdate={broadcastToOthers}
  />
</Whiteboard>
```

---

## 8. Implementation Roadmap

### Phase 1: Basic Video Conferencing (Sprints 1–3, ~6 weeks)

**Sprint 1:**
- [ ] WebRTC setup (simple-peer + Socket.io)
- [ ] Database schema (ClassroomSession, Participant, AttendanceLog)
- [ ] Session creation & join flow
- [ ] Basic video/audio (no screen share yet)

**Sprint 2:**
- [ ] Teacher controls (mute, remove, admit)
- [ ] Participant list UI
- [ ] Chat system (public)
- [ ] Room code system

**Sprint 3:**
- [ ] Attendance auto-logging
- [ ] Session recording (basic)
- [ ] Recording playback
- [ ] Mobile optimization (starts here)

### Phase 2: Interactive Features (Sprints 4–5, ~4 weeks)

**Sprint 4:**
- [ ] Screen sharing
- [ ] Raise hand system
- [ ] Polls (real-time voting)
- [ ] Whiteboard (Excalidraw integration)

**Sprint 5:**
- [ ] Breakout rooms
- [ ] Private chat (student to teacher)
- [ ] Chat moderation (pin, delete, hide)
- [ ] Engagement metrics (poll results, hand raises)

### Phase 3: Integration & Recording (Sprints 6–7, ~4 weeks)

**Sprint 6:**
- [ ] Recording quality optimization
- [ ] Auto-publish recordings to Q Social
- [ ] Recording retention policies
- [ ] Teacher recording management dashboard

**Sprint 7:**
- [ ] Attendance sync to Progress records
- [ ] Parent notification ("Your child attended class")
- [ ] Attendance report (for teachers)
- [ ] COPPA consent for recording

### Phase 4: Teacher Dashboard & Admin Tools (Sprint 8, ~2 weeks)

**Sprint 8:**
- [ ] Classroom scheduling UI
- [ ] Session history & reporting
- [ ] Recording library management
- [ ] Class settings (defaults for recording, chat, etc.)

### Phase 5: Launch Prep (Sprints 9–10, ~3 weeks)

**Sprint 9:**
- [ ] UAT with pilot teachers
- [ ] Performance testing (10+ concurrent users)
- [ ] Accessibility audit (WCAG 2.1)
- [ ] Security testing (WebRTC attack vectors)

**Sprint 10:**
- [ ] Documentation (teacher & student guides)
- [ ] Teacher training materials
- [ ] Parent communication
- [ ] Bug fixes & polish

---

## 9. Privacy & Compliance

### 9.1 Recording Consent (COPPA/GDPR-K)

```typescript
// Before class starts:
if (session.hasMinors) {
  // Require parental consent
  await requireGuardianConsentForRecording(studentIds);
}

// Teacher must confirm:
- [ ] "I have notified all guardians this class will be recorded"
- [ ] "I have parental consent for recording this session"
- [ ] "Recording will be stored securely and only shared with authorized users"

// Recording consent metadata:
{
  recordingConsent: {
    requiredStudents: ["student_id_1", "student_id_2"],
    consentGivenBy: ["guardian_id_1", "guardian_id_2"],
    consentedAt: "2026-09-16T10:00:00Z",
    consentVersion: "1.0"
  }
}
```

### 9.2 Attendance Tracking (FERPA)

```
Rule: Student attendance is educational record (FERPA-protected)
├── Only accessible to: Student, parents, authorized school staff
├── Not shared with: Public, other students (except aggregate stats)
└── Retention: Follows school retention policy (typically 7 years)

Aggregate Stats (OK to share):
├── "Class attendance average: 92%"
├── "On-time percentage by student: ..."
└── But NOT individual student details without authorization
```

### 9.3 Chat & Data Retention

```
Chat Messages:
├── Kept during class: Real-time visible
├── After class ends:
│   ├── If public: Archive for 6 months (searchable)
│   ├── If private: Archive for 1 year (between participants only)
│   └── Auto-delete after retention period
│
└── Searchable by: Teacher (own class), admin, parent (if child authorized)
```

---

## 10. Key Metrics & Success Criteria

### For V0.95 Pilot

- **Adoption:** ≥50% of teachers schedule ≥1 class in 4-week pilot
- **Session Quality:** ≥95% of sessions complete without major technical issues
- **Attendance Accuracy:** >99% accuracy in attendance logging vs. manual records
- **User Satisfaction:** ≥4/5 for both teachers and students
- **Recording Success:** ≥95% of sessions record successfully
- **Performance:** Video latency <200ms, audio latency <100ms

### For V1.0 Launch

- **Monthly Active Sessions:** ≥1,000 sessions/month across pilot schools
- **Recording Utilization:** ≥40% of students watch recorded classes
- **Engagement:** Average 3+ poll responses per class, 5+ messages per session
- **Teacher Satisfaction:** ≥4.5/5 on feature completeness
- **Technical Uptime:** ≥99.5% availability during school hours
- **Zero Privacy Incidents:** No unauthorized recording access or data breaches

---

## 11. Architecture Comparison: DIY vs. Integrate

### Option A: Use BigBlueButton Server (Recommended for V0.95)

**Pros:**
- All features included
- Proven, production-ready
- Active community support
- Focus UI customization only

**Cons:**
- Requires separate server infrastructure
- Heavyweight (~8GB RAM minimum)
- Licensing complexity (AGPL)

**Effort:** 4–6 weeks (custom Next.js UI + integration layer)

---

### Option B: Build Custom with Jitsi (Mid-weight)

**Pros:**
- React-based (your stack)
- Customizable UI
- Lightweight (~2GB RAM)

**Cons:**
- Missing some enterprise features (e.g., recording)
- Need to build/integrate recording separately
- More development effort

**Effort:** 8–10 weeks (custom recording, whiteboard, breakout rooms)

---

### Option C: Lightweight Custom (WebRTC + Socket.io)

**Pros:**
- Full control
- Minimal dependencies
- Best performance (if done right)

**Cons:**
- Everything from scratch (recording, encoding, etc.)
- High development risk
- Requires DevOps expertise

**Effort:** 12–16 weeks (highest risk)

---

## 12. Recommendation: Option A (BigBlueButton) for MVP

**Why:**
1. Reduces risk (proven platform)
2. Faster to launch (focus on integration, not building)
3. Feature-complete (no missing features)
4. Allows team to focus on Q-specific customizations

**Proposed Architecture:**

```
BigBlueButton Server (Hosted)
    ↓
   REST API
    ↓
NestJS Backend Layer
├── Session management
├── Attendance sync
├── Recording metadata
└── Q ecosystem integration
    ↓
Next.js Frontend
├── Custom classroom UI
├── Attendance dashboard
└── Integration with Q Social/Progress
```

---

## 13. Resource Requirements

### Development Team

- **Backend Developer:** 1 FTE (BigBlueButton integration, NestJS API)
- **Frontend Developer:** 1.5 FTE (Custom UI, real-time updates)
- **DevOps/Infrastructure:** 0.5 FTE (BBB server setup, monitoring)
- **QA/Testing:** 0.5 FTE (UAT, performance testing)
- **Product Manager:** 0.25 FTE (oversight)

**Total:** ~3.75 FTE for 10 sprints

### Infrastructure (BigBlueButton)

- **BigBlueButton Server:** 8GB RAM, 4 CPU (AWS: ~$150–$200/month)
- **Recording Storage:** AWS S3 ($50–$200/month depending on video volume)
- **CDN for Recordings:** Cloudflare ($20–$50/month)
- **Monitoring & Logging:** CloudWatch/DataDog ($50–$100/month)

**Total Infrastructure:** ~$300–$400/month

---

## 14. Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| **Video Latency** | Medium | Medium | Proper TURN server, bandwidth monitoring |
| **Recording Failures** | Low | High | Robust error handling, manual re-record option |
| **Privacy Breaches** | Low | Critical | Encryption, access controls, regular audits |
| **Server Overload** | Medium | High | Load balancing, auto-scaling, capacity planning |
| **COPPA Violations** | Low | Critical | Strict consent flows, legal review |
| **User Adoption** | Medium | Medium | Teacher training, gradual rollout |

---

## 15. Next Steps

1. **Decide on architecture:** BigBlueButton (recommended) vs. Custom
2. **Get stakeholder approval:**
   - [ ] IT/Infrastructure team (server capacity)
   - [ ] Legal team (COPPA, FERPA compliance)
   - [ ] Teachers (feature requirements)
   - [ ] Parents (privacy concerns)
3. **Allocate resources** and assign sprint owners
4. **Create GitHub issues** for each sprint
5. **Set up infrastructure** (BigBlueButton server if chosen)

---

## Appendix A: Reference Repository List

| Component | Repository | Stars | Language | Use Case |
|-----------|------------|-------|----------|----------|
| Live Classroom (Full) | [bigbluebutton/bigbluebutton](https://github.com/bigbluebutton/bigbluebutton) | 8K | Java/React | Production-ready LMS integration |
| Video Conferencing | [jitsi/jitsi-meet](https://github.com/jitsi/jitsi-meet) | 22K | React/JavaScript | Lightweight alternative |
| WebRTC Classroom | [Hashim-stack/samvaad-webrtc-classroom](https://github.com/Hashim-stack/samvaad-webrtc-classroom) | 300 | Next.js/Node | Custom implementation reference |
| Whiteboard | [excalidraw/excalidraw](https://github.com/excalidraw/excalidraw) | 66K | React/TypeScript | Real-time collaborative whiteboard |
| Virtual Classroom | [suvojitmanna/Live-classes](https://github.com/suvojitmanna/Live-classes) | 100 | MERN | Full-stack reference |
| Streaming | [owncast/owncast](https://github.com/owncast/owncast) | 8K | Go | Live recording & streaming |

---

**Document Status:** READY FOR REVIEW  
**Last Updated:** September 16, 2026  
**Owner:** GitHub Copilot  
**Recommended Decision:** BigBlueButton-based architecture  
**Target Launch:** V1.0 or V0.95 Pilot Phase

