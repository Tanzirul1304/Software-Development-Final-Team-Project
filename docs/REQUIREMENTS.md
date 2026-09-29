# Requirements.md

## Context
When a university student is waiting on an instructor's answer to a course
question but replies on Moodle or email take ~12 hours, our communication-only
channel lets the instructor see and answer student messages without opening
email or logging into Moodle, so that average response time goes from 12
hours to 2 hours.

Out of scope for now: built-in video meetings (students/teachers keep using
Zoom/Google Meet), grading/gradebook and assignment submission, and course
content/file hosting. Audio may be added later.

Stack: Supabase (database), Netlify (hosting/dev URL). Class meets every
Monday.

---

## 1. Functional requirements

1. A student can compose and send a message tagged to a specific course (and
   optionally an assignment) to the course instructor.
2. An instructor receives an in-app notification when a new message arrives,
   without needing email or Moodle.
3. A message thread shows status: sent, seen, replied.
4. A student is notified when the instructor replies.
5. An instructor can mark a thread as resolved.
6. A teaching assistant can be added to a course's threads to help answer
   questions when the instructor is unavailable.
7. An instructor can set an availability status (available/away), visible to
   students before they send a message.
8. A student can search past message threads by course or keyword.
9. If a message goes unanswered for 2 hours, the student can flag it as
   urgent.

---

## 2. Non-functional requirements

| Category | Target | How we will check it |
|---|---|---|
| Performance | First screen (inbox) usable within 2 seconds on 4G. | Chrome DevTools "Fast 4G" throttling profile, tested on a team laptop against the Netlify dev URL; record Time to Interactive from Lighthouse, with device/browser and date. |
| Accessibility | Main tasks (login, compose/send a message, read inbox) work by keyboard only; axe reports 0 serious issues. | Tab through each task with no mouse; run axe DevTools on each page; record page, task, keyboard result, axe serious-issue count, and date. |
| Security | Row Level Security enabled on every Supabase table; a test user cannot access another user's private records. | For each table, sign in as Test User A and attempt to read/write Test User B's records; confirm 0 rows / permission denied. Record table name, RLS status, test result, date. |
| Privacy | 0 real personal-data fields in client records used for the app, prompts, or repository. | Fields stored: name, student ID, course name (see justification below). All seed/test data is fictional. Grep the repo for real names/emails before each commit and demo. |
| Availability | Dev URL works during all class hours (every Monday); same-day rollback after a failed deployment. | Visit the dev URL at the start of each Monday class; log status and timestamp. If a deploy breaks the app, roll back to the last working Netlify deploy the same day and log the incident and resolution time. |

Personal-data fields and why the app needs them:
- **Name** — identifies sender/recipient in a thread.
- **Student ID** — matches a message to a course roster/enrollment record.
- **Course name** — routes and tags the message to the right class and instructor.

---

## 3. User stories

### Story 1 — Ask a tagged question
As a student, I want to send a message tagged to a specific course, so that
my question has context and reaches the right instructor quickly.

- Given a student is enrolled in a course and viewing it in the app,
  when they compose a message and tag it to that course,
  then the message is delivered to the instructor's inbox with the course
  reference attached, and the student sees a "sent" status.

### Story 2 — Instant instructor notification
As an instructor, I want an instant notification when a student messages me,
so that I can respond without checking email or Moodle.

- Given an instructor has notifications enabled in the app,
  when a student sends them a new message,
  then the instructor receives a notification within 10 seconds containing
  sender name, course, and a message preview, with no email or Moodle step
  required.

### Story 3 — Message status visibility
As a student, I want to see whether my message has been read or replied to,
so that I know whether to expect an answer or should follow up.

- Given a student has sent a message to an instructor,
  when the instructor opens and reads it,
  then the student's thread updates to a "seen" state within 10 seconds, and
  if there is no reply after 2 hours, the student is offered an option to
  flag the message as urgent.

### Story 4 — Mark thread resolved
As an instructor, I want to mark a thread resolved, so my inbox only shows
conversations still needing a response.

- Given an instructor is viewing an open thread with a student,
  when the instructor marks the thread as resolved,
  then the thread moves out of the active inbox into a resolved list, and the
  student can still see the full history.

### Story 5 — Reply notification
As a student, I want a notification when my instructor replies, so I don't
have to keep re-checking the app.

- Given a student has an open thread with an instructor,
  when the instructor sends a reply,
  then the student receives a notification within 10 seconds naming the
  course and a preview of the reply.

### Story 6 — Teaching assistant looped in
As a teaching assistant, I want to be looped into a course's threads, so I
can help answer questions when the instructor is unavailable.

- Given a TA has been added to a course by the instructor,
  when a student sends a message in that course,
  then the TA sees the message in their inbox alongside the instructor, and
  can reply on the instructor's behalf.

### Story 7 — Availability status
As an instructor, I want to set an "available" / "away" status, so students
know whether to expect a fast or delayed reply.

- Given an instructor opens their status control,
  when they set their status to "away",
  then students composing a new message to that instructor see an
  "away, replies may be delayed" notice before sending.

### Story 8 — Search past threads
As a student, I want to search past threads by course or keyword, so I can
find an answer without re-asking.

- Given a student has one or more past message threads,
  when they enter a course name or keyword into the search bar,
  then matching threads are listed, ranked by relevance/recency, within
  2 seconds.

---

## 4. Events

Client task: a student asks a question and gets a timely instructor reply.

1. Command: Compose message (student) → Event: Message was composed.
2. Command: Send message (student) → Event: Message was sent.
3. Command: Deliver notification (system, on message sent) → Event: Instructor was notified.
4. Command: Open thread (instructor) → Event: Message was read.
5. Command: Send reply (instructor) → Event: Reply was sent.
6. Command: Deliver notification (system, on reply sent) → Event: Student was notified.
7. Command: Open thread (student) → Event: Reply was read.
8. Command: Mark thread resolved (instructor) → Event: Thread was resolved.

---

## 5. Milestones

1. Requirements gate (G2) closed: functional/non-functional requirements,
   eight user stories, events and milestones finalized and posted to Moodle.
2. Core messaging live: student-to-instructor tagged messaging deployed to
   the Netlify dev URL with Row Level Security enabled on all Supabase
   tables.
3. Notifications and status complete: in-app notifications, seen/replied
   status, resolved/away status shipped; keyboard and axe accessibility
   checks pass on core pages.
4. Automated coverage (G3 prep): Playwright tests written for all eight
   stories and passing.
5. Pilot-ready release: privacy check passed (fictional data only), dated
   availability logs show no unresolved downtime across class hours, ready
   to pilot with a real course section.
