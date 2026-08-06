# UMBC Neo4j Academic Graph Database - HackUMBC Edition

## Welcome HackUMBC Participants!

Welcome to the HackUMBC Data Science track! Your mission is to leverage this rich synthetic dataset of UMBC student data to create something amazing. The possibilities are endless:

### **Project Ideas & Directions**

#### **Build an Application**
- **Student Advisor Dashboard**: Create a web app that visualizes optimal course paths, identifies at-risk students, or recommends study groups based on learning styles
- **Faculty Analytics Tool**: Help professors understand course difficulty patterns, prerequisite effectiveness, or textbook usage
- **Degree Planning Assistant**: Build an interactive tool that shows students the fastest path to graduation considering their learning style and course history

#### **Extend the Dataset**
Want to take the data in a new direction? Modify `generate_synthetic_dataset.py`:
- **Add New Node Types** (lines 255-507): Add internships, clubs, research projects, or campus events
- **Create New Relationships** (lines 652-1406): Model advisor relationships, study groups, or research collaborations  
- **Modify Data Distributions** (lines 48-91): Change learning styles, grade distributions, or difficulty patterns
- **Add New Departments** (lines 79-82): Expand beyond CS and Biology
- **Enhance Student Profiles** (lines 255-339): Add GPA, interests, career goals, or extracurriculars
- **Generate Time-Series Data** (lines 1320-1406): Add attendance patterns, assignment submissions, or office hour visits

#### **Data Analysis & Export**
- Run graph algorithms to find influential courses or optimal degree paths
- Export data for machine learning models predicting student success
- Create visualizations of department interconnections
- Analyze textbook effectiveness vs. student performance

#### **Integration Projects**
- Connect the graph to external APIs (Canvas, course catalogs, rate my professor)
- Build a recommendation engine for course selection
- Create a chatbot that queries the graph to answer student questions

### **Judging Criteria Considerations**
Your project could excel in:
- **Innovation**: Novel use of graph relationships (similarity networks, prerequisite chains)
- **Impact**: Solving real problems for students, faculty, or administrators
- **Technical Excellence**: Efficient Cypher queries, graph algorithms, clean architecture
- **Visualization**: Making complex academic relationships understandable
- **Scalability**: Solutions that work with larger datasets

### **Quick Start**
1. Follow the setup instructions below to get Neo4j running with the dataset
2. Explore the data model and example queries
3. Identify your target users (students, faculty, advisors, administrators)
4. Build something awesome!

Remember: You're not limited to just querying - you can extend the generator, export data, or build full applications!

---

## Setup Instructions

A comprehensive synthetic academic dataset for UMBC (University of Maryland, Baltimore County), designed for modeling student degree pathways and educational analytics in Neo4j. This dataset focuses on Computer Science and Biology departments with rich relationships between students, courses, faculty, textbooks, and learning patterns.

## Quick Start for Hackathon Participants

### Prerequisites
- Python 3.8+ 
- Neo4j Desktop or Neo4j Community Edition 5.x
- 8GB+ RAM recommended
- 2GB free disk space

## Step 1: Install Neo4j

### Option A: Neo4j Desktop (Recommended for Hackathon)
1. Download Neo4j Desktop from https://neo4j.com/download/
2. Install and launch Neo4j Desktop
3. Create a new project called "UMBC Academic"
4. Add a new database (Neo4j 5.x):
   - Click "Add" → "Local DBMS"
   - Name: "UMBC Academic Database"
   - Password: Set a password (remember it!)
   - Version: 5.x (latest)
5. Install APOC plugin (REQUIRED):
   - Click on your database
   - Go to "Plugins" tab
   - Find APOC and click "Install"
6. Install Graph Data Science plugin (OPTIONAL but recommended):
   - In the Plugins tab, find "Graph Data Science Library"
   - Click "Install"
7. Start the database

### Option B: Neo4j Community Edition (Docker)
```bash
# Pull and run Neo4j with APOC
docker run -d \
  --name umbc-neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/yourpassword \
  -e NEO4J_PLUGINS='["apoc", "graph-data-science"]' \
  -e NEO4J_apoc_export_file_enabled=true \
  -e NEO4J_apoc_import_file_enabled=true \
  -e NEO4J_apoc_import_file_use__neo4j__config=true \
  -v $PWD/import:/var/lib/neo4j/import \
  neo4j:5-community
```

## Step 2: Generate the Synthetic Dataset

### Setup Python Environment
```bash
# Clone or download this repository
git clone https://github.com/jasonpaluck/hackumbc-2025.git
cd hackumbc-2025

# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate  # On macOS/Linux
# OR
venv\Scripts\activate     # On Windows

# Install requirements
pip install -r requirements.txt
```

### Generate Data
```bash
# Run the generator
python3 generate_synthetic_dataset.py

# This creates the umbc_data/ directory with:
# - CSV files for bulk import
# - Cypher scripts for direct import
# - README with data dictionary
# - Browser guide with example queries
```

## Step 3: Import Data into Neo4j

### Method 1: Using Cypher Scripts (Easiest for Hackathon)

⚠️ **IMPORTANT**: The Cypher import process takes significant time due to the thousands of MATCH operations required to create relationships. Plan accordingly and start early!

1. Open Neo4j Browser (http://localhost:7474)
2. Login with your credentials
3. Run these commands in order:

```cypher
// First, clear any existing data (optional)
MATCH (n) DETACH DELETE n;

// Create constraints and indexes (REQUIRED - run first!)
CREATE CONSTRAINT student_id IF NOT EXISTS FOR (s:Student) REQUIRE s.id IS UNIQUE;
CREATE CONSTRAINT course_id IF NOT EXISTS FOR (c:Course) REQUIRE c.id IS UNIQUE;
CREATE CONSTRAINT faculty_id IF NOT EXISTS FOR (f:Faculty) REQUIRE f.id IS UNIQUE;
CREATE CONSTRAINT degree_id IF NOT EXISTS FOR (d:Degree) REQUIRE d.id IS UNIQUE;
CREATE CONSTRAINT requirement_id IF NOT EXISTS FOR (r:RequirementGroup) REQUIRE r.id IS UNIQUE;
CREATE CONSTRAINT term_id IF NOT EXISTS FOR (t:Term) REQUIRE t.id IS UNIQUE;
CREATE CONSTRAINT textbook_id IF NOT EXISTS FOR (tb:Textbook) REQUIRE tb.id IS UNIQUE;

// Create indexes for better query performance
CREATE INDEX student_learning_style IF NOT EXISTS FOR (s:Student) ON (s.learningStyle);
CREATE INDEX course_department IF NOT EXISTS FOR (c:Course) ON (c.department);
CREATE INDEX course_level IF NOT EXISTS FOR (c:Course) ON (c.level);
CREATE INDEX term_type IF NOT EXISTS FOR (t:Term) ON (t.type);
```

4. Import the Cypher files in order from `umbc_data/cypher/`:
   - Files 01-07: Node creation (fast)
   - Files 08-15: Basic relationships
   - Files 16-21: Large relationship sets (thousands of MATCH operations each)

💡 **Pro Tips**: 
- Start the import at the beginning of the hackathon while you plan your project
- You can skip the textbook interaction files (20-21) if you're not analyzing reading patterns
- Consider importing only a subset of the completed courses if you need faster setup

### Reducing Dataset Size (If Needed)

If you're running into memory/time constraints, modify these parameters in `generate_synthetic_dataset.py` before generating:

```python
# Line 34-40 - Reduce these values:
NUM_STUDENTS = 100        # Reduce from 500
NUM_COURSES = 50          # Reduce from 100  
AVG_COURSES_PER_STUDENT = 10  # Reduce from 20

# This will generate a smaller but still functional dataset
```

After modifying, regenerate the dataset:
```bash
python3 generate_synthetic_dataset.py
```

A smaller dataset will import much faster while still providing rich relationship patterns for your project!

### Method 2: Using CSV Import (For Large Datasets)
```bash
# Copy CSV files to Neo4j import directory
# For Neo4j Desktop: Find import directory in database settings
# For Docker: Already mapped to ./import

# Run this Cypher command in Neo4j Browser:
:auto USING PERIODIC COMMIT 1000
LOAD CSV WITH HEADERS FROM 'file:///students.csv' AS row
CREATE (s:Student {
  id: row.id,
  name: row.name,
  enrollmentDate: date(row.enrollmentDate),
  expectedGraduation: date(row.expectedGraduation),
  learningStyle: row.learningStyle,
  preferredCourseLoad: toInteger(row.preferredCourseLoad),
  preferredPace: row.preferredPace,
  riskTolerance: toFloat(row.riskTolerance),
  careerGoal: row.careerGoal
});

// Continue with other CSV files...
```

## Step 4: Verify Import

Run this query to check your data:
```cypher
// Check node counts
MATCH (n) RETURN labels(n)[0] as Label, count(n) as Count;

// Should see approximately:
// Student: 500
// Course: 100
// Faculty: 30
// Degree: 4
// Term: 12
// Textbook: 80-100
```

## Step 5: View Browser Guide

1. Open the HTML file `umbc_data/umbc_guide.html` for additional insights.

## Data Model

The database follows this graph model:

- **Nodes**: Student, Course, Faculty, Degree, RequirementGroup, Term, Textbook
- **Relationships**:
  - Student-Course: COMPLETED, ENROLLED_IN
  - Student-Degree: PURSUING
  - Student-Student: SIMILAR_LEARNING_STYLE, SIMILAR_PERFORMANCE
  - Course-Course: PREREQUISITE_FOR, LEADS_TO, SIMILAR_CONTENT, SIMILAR_DIFFICULTY
  - Faculty-Course: TEACHES
  - RequirementGroup-Degree: PART_OF
  - Course-RequirementGroup: FULFILLS
  - Course-Term: OFFERED_IN
  - Course-Textbook: REQUIRES, RECOMMENDS
  - Student-Textbook: VIEWED_PAGE, INTERACTED_WITH

## 🔗 Student Similarity Relationships - The Key to Behavioral Insights

The dataset includes two powerful student similarity relationships that enable peer-based analytics:

### **SIMILAR_LEARNING_STYLE** Relationship
- Connects students with matching learning styles (Visual, Auditory, Kinesthetic, Reading-Writing)
- Similarity score (0.1-1.0) based on:
  - Learning style match (base 0.7)
  - Preferred pace match (+0.1)
  - Instruction mode preference (+0.1)
  - Course load similarity (adjusted)

### **SIMILAR_PERFORMANCE** Relationship  
- Connects students who took 3+ common courses
- Similarity score (0.1-1.0) calculated from:
  - 70% weight: Grade similarity across common courses
  - 30% weight: Difficulty perception alignment
- Includes list of common courses in the relationship

### **The Power of Similarity: Behavioral Change Recommendations**

**Core Insight Question**: *"Students most similar to you who are performing better are behaving differently. What specific changes could you make to improve your academic performance?"*

This is the foundation for powerful recommendation systems. Here's how to approach it:

### Example: Comprehensive Behavioral Analysis Query

```cypher
// Find what successful similar students do differently
// First get a sample student who has course completions
MATCH (target:Student)-[:COMPLETED]->(:Course)
WITH target.id AS targetStudentId LIMIT 1  // Get first student with courses

// Get target student's GPA equivalent
MATCH (target:Student {id: targetStudentId})-[tc:COMPLETED]->(c:Course)
WITH target, targetStudentId, 
     AVG(CASE tc.grade
         WHEN 'A' THEN 4.0 WHEN 'A-' THEN 3.7
         WHEN 'B+' THEN 3.3 WHEN 'B' THEN 3.0 WHEN 'B-' THEN 2.7
         WHEN 'C+' THEN 2.3 WHEN 'C' THEN 2.0 WHEN 'C-' THEN 1.7
         WHEN 'D+' THEN 1.3 WHEN 'D' THEN 1.0
         ELSE 0 END) AS targetGPA

// Find similar students who are performing better
MATCH (target)-[sim:SIMILAR_LEARNING_STYLE|SIMILAR_PERFORMANCE]->(peer:Student)
WHERE sim.similarity > 0.7
MATCH (peer)-[pc:COMPLETED]->(course:Course)
WITH target, targetGPA, peer, sim,
     AVG(CASE pc.grade
         WHEN 'A' THEN 4.0 WHEN 'A-' THEN 3.7
         WHEN 'B+' THEN 3.3 WHEN 'B' THEN 3.0 WHEN 'B-' THEN 2.7
         WHEN 'C+' THEN 2.3 WHEN 'C' THEN 2.0 WHEN 'C-' THEN 1.7
         WHEN 'D+' THEN 1.3 WHEN 'D' THEN 1.0
         ELSE 0 END) AS peerGPA
WHERE peerGPA > targetGPA + 0.3  // At least 0.3 GPA points higher

// Analyze behavioral differences in multiple dimensions
MATCH (peer)-[peerCompleted:COMPLETED]->(course:Course)

// 1. Course Load Analysis
WITH target, peer, 
     COUNT(DISTINCT peerCompleted) AS peerCourseCount,
     AVG(peerCompleted.difficulty) AS peerPerceivedDifficulty,
     COLLECT(DISTINCT course.level) AS peerCourseLevels,
     peerGPA

// 2. Textbook Engagement Analysis
OPTIONAL MATCH (peer)-[peerView:VIEWED_PAGE]->(textbook:Textbook)
WITH target, peer, peerGPA, peerCourseCount, peerPerceivedDifficulty, peerCourseLevels,
     COUNT(peerView) AS peerTextbookViews,
     COUNT(DISTINCT textbook) AS peerTextbooksUsed,
     AVG(peerView.duration) AS peerAvgReadingDuration

// 3. Study Pattern Analysis
OPTIONAL MATCH (peer)-[peerInt:INTERACTED_WITH]->(textbook2:Textbook)
WHERE peerInt.interactionType IN ['highlight', 'note']
WITH target, peer, peerGPA, peerCourseCount, peerPerceivedDifficulty, 
     peerCourseLevels, peerTextbookViews, peerTextbooksUsed, peerAvgReadingDuration,
     COUNT(peerInt) AS peerActiveStudyInteractions

// Get target student's behaviors for comparison
MATCH (target)-[targetCompleted:COMPLETED]->(tCourse:Course)
OPTIONAL MATCH (target)-[targetView:VIEWED_PAGE]->(tTextbook:Textbook)
OPTIONAL MATCH (target)-[targetInt:INTERACTED_WITH]->(tTextbook2:Textbook)
WHERE targetInt.interactionType IN ['highlight', 'note']

WITH peer, peerGPA,
     // Peer metrics
     peerCourseCount, peerPerceivedDifficulty, peerTextbookViews, 
     peerTextbooksUsed, peerAvgReadingDuration, peerActiveStudyInteractions,
     // Target metrics
     COUNT(DISTINCT targetCompleted) AS targetCourseCount,
     AVG(targetCompleted.difficulty) AS targetPerceivedDifficulty,
     COUNT(targetView) AS targetTextbookViews,
     COUNT(DISTINCT tTextbook) AS targetTextbooksUsed,
     AVG(targetView.duration) AS targetAvgReadingDuration,
     COUNT(targetInt) AS targetActiveStudyInteractions

// Calculate behavioral differences and generate recommendations
RETURN 
    // Performance Gap
    ROUND(AVG(peerGPA - targetGPA), 2) AS gpaGap,
    COUNT(DISTINCT peer) AS similarSuccessfulPeers,
    
    // Course Load Insights
    ROUND(AVG(peerCourseCount - targetCourseCount)) AS coursesPerTermDifference,
    CASE 
        WHEN AVG(peerCourseCount) > AVG(targetCourseCount) 
        THEN 'Take ' + toString(ROUND(AVG(peerCourseCount - targetCourseCount))) + ' more courses per term'
        WHEN AVG(peerCourseCount) < AVG(targetCourseCount)
        THEN 'Reduce course load by ' + toString(ROUND(AVG(targetCourseCount - peerCourseCount))) + ' courses'
        ELSE 'Course load is optimal'
    END AS courseLoadRecommendation,
    
    // Textbook Engagement Insights
    ROUND(AVG(peerTextbookViews - targetTextbookViews)) AS textbookEngagementGap,
    CASE 
        WHEN AVG(peerTextbookViews) > AVG(targetTextbookViews) * 1.5
        THEN 'Increase textbook reading by ' + toString(ROUND((AVG(peerTextbookViews) / AVG(targetTextbookViews) - 1) * 100)) + '%'
        ELSE 'Textbook engagement is sufficient'
    END AS textbookRecommendation,
    
    // Study Behavior Insights
    ROUND(AVG(peerAvgReadingDuration - targetAvgReadingDuration), 1) AS readingDurationGap,
    CASE 
        WHEN AVG(peerAvgReadingDuration) > AVG(targetAvgReadingDuration)
        THEN 'Spend ' + toString(ROUND(AVG(peerAvgReadingDuration - targetAvgReadingDuration))) + ' more minutes per reading session'
        ELSE 'Reading session duration is good'
    END AS readingDurationRecommendation,
    
    // Active Learning Insights
    ROUND(AVG(peerActiveStudyInteractions - targetActiveStudyInteractions)) AS activeStudyGap,
    CASE 
        WHEN AVG(peerActiveStudyInteractions) > AVG(targetActiveStudyInteractions) * 1.5
        THEN 'Increase active study (highlighting/notes) by ' + toString(ROUND((AVG(peerActiveStudyInteractions) / NULLIF(AVG(targetActiveStudyInteractions), 0) - 1) * 100)) + '%'
        ELSE 'Active study engagement is adequate'
    END AS activeStudyRecommendation,
    
    // Difficulty Perception
    ROUND(AVG(targetPerceivedDifficulty - peerPerceivedDifficulty), 2) AS difficultyPerceptionGap,
    CASE
        WHEN AVG(targetPerceivedDifficulty) > AVG(peerPerceivedDifficulty) + 0.5
        THEN 'Similar students find courses easier - consider seeking study groups or tutoring'
        ELSE 'Difficulty perception aligns with successful peers'
    END AS difficultyInsight
```

### Simpler Targeted Queries

**Find Study Buddies Who Succeeded in Your Struggling Course:**
```cypher
// Find peers who aced a course you're struggling with
// First find a student with a low grade in any course
MATCH (you:Student)-[yourGrade:COMPLETED]->(course:Course)
WHERE yourGrade.grade IN ['C', 'C-', 'D', 'F']
WITH you, course, yourGrade LIMIT 1  // Get first example
MATCH (you)-[sim:SIMILAR_LEARNING_STYLE]->(peer:Student)-[peerGrade:COMPLETED]->(course)
WHERE sim.similarity > 0.8 
  AND peerGrade.grade IN ['A', 'A-', 'B+']
RETURN peer.id, peer.name, peerGrade.grade, sim.similarity,
       peerGrade.studyHours - yourGrade.studyHours AS extraStudyHours
ORDER BY sim.similarity DESC
```

**Identify Optimal Course Combinations Based on Similar Students:**
```cypher
// What course combinations work well for students like you?
// First get a student with similar learning style connections
MATCH (you:Student)-[:SIMILAR_LEARNING_STYLE]->(peer:Student)
WITH you LIMIT 1  // Get first student with similarity connections
MATCH (you)-[sim:SIMILAR_LEARNING_STYLE]->(peer:Student)
WHERE sim.similarity > 0.75
MATCH (peer)-[c1:COMPLETED]->(course1:Course), 
      (peer)-[c2:COMPLETED]->(course2:Course)
WHERE c1.term = c2.term  // Taken in same term
  AND c1.grade IN ['A', 'A-', 'B+']
  AND c2.grade IN ['A', 'A-', 'B+']
WITH course1, course2, COUNT(DISTINCT peer) AS successfulPeers
WHERE successfulPeers >= 3
RETURN course1.name, course2.name, successfulPeers
ORDER BY successfulPeers DESC
```

These similarity relationships transform the dataset from static academic records into a dynamic recommendation engine that can provide personalized, evidence-based advice for student success!

## Example Queries and Use Cases

### 1. Find Optimal Next Courses for a Student

```cypher
// Find optimal next courses based on learning style and prerequisites
// First, get a sample student and current term
MATCH (student:Student)-[:PURSUING]->(degree:Degree)
WITH student LIMIT 1  // Get first student, or replace with specific ID
MATCH (term:Term)
WHERE term.startDate > date()  // Get future term
WITH student, term, degree ORDER BY term.startDate LIMIT 1
MATCH (course:Course)-[:OFFERED_IN]->(term)
MATCH (course)-[:FULFILLS]->(:RequirementGroup)-[:PART_OF]->(degree)

// Ensure prerequisites are met
WHERE NOT EXISTS {
  MATCH (prereq:Course)-[:PREREQUISITE_FOR]->(course)
  WHERE NOT (student)-[:COMPLETED]->(prereq)
}

// Student hasn't already completed the course
AND NOT (student)-[:COMPLETED]->(course)

// Find similar students and their experiences with these courses
OPTIONAL MATCH (student)-[sim:SIMILAR_LEARNING_STYLE]->(similar:Student)-[comp:COMPLETED]->(course)
WHERE sim.similarity > 0.7

WITH student, course, term, degree,
     CASE WHEN COUNT(comp) > 0 
          THEN AVG(comp.difficulty) 
          ELSE course.avgDifficulty END AS predictedDifficulty,
     COUNT(comp) AS similarStudentsCount

// Check how many future courses this would unlock
OPTIONAL MATCH (course)-[:PREREQUISITE_FOR]->(futureCourse)
WHERE (futureCourse)-[:FULFILLS]->(:RequirementGroup)-[:PART_OF]->(degree)

RETURN course.id AS courseId,
       course.name AS courseName,
       course.credits AS credits,
       predictedDifficulty,
       COUNT(futureCourse) AS unlockedCourses,
       similarStudentsCount,
       course.instructionModes AS availableModes
ORDER BY predictedDifficulty ASC, unlockedCourses DESC, credits DESC
LIMIT 5
```

**Use Case**: Course Planning
- Helps students select courses that align with their learning style
- Considers prerequisite chains to optimize degree progress
- Predicts difficulty based on similar students' experiences
- Shows how many future courses each option unlocks

**Graph DB Advantage**: 
- Easily traverses prerequisite chains without complex joins
- Efficiently finds similar students and their course experiences
- Natural representation of course relationships and dependencies

### 2. Analyze Student Reading Patterns

```cypher
// Compare reading patterns between high and low performing students
MATCH (student:Student)-[comp:COMPLETED]->(course:Course)
MATCH (student)-[view:VIEWED_PAGE]->(textbook:Textbook)<-[:REQUIRES]-(course)
WITH student, course, textbook, comp,
     COUNT(view) AS totalPages,
     AVG(view.duration) AS avgTimePerPage,
     COLLECT(view.timestamp) AS viewTimes
WITH student, course, textbook,
     totalPages,
     avgTimePerPage,
     // Calculate reading pattern metrics
     CASE WHEN size(viewTimes) > 1
          THEN reduce(s = 0, i in range(0, size(viewTimes)-2) |
               s + duration.between(
                    datetime(replace(viewTimes[i], ' ', 'T')),
                    datetime(replace(viewTimes[i+1], ' ', 'T'))
               ).days)
          ELSE 0 END AS totalGaps,
     size(viewTimes) AS totalSessions,
     comp.grade AS grade
WHERE comp.grade IN ['A', 'A-', 'B+', 'C', 'C-', 'D']
RETURN student.id,
       course.name,
       grade,
       totalPages,
       avgTimePerPage,
       totalGaps / totalSessions AS avgDaysBetweenSessions,
       CASE 
         WHEN grade IN ['A', 'A-', 'B+'] THEN 'High Performing'
         ELSE 'Low Performing'
       END AS performanceCategory
ORDER BY grade ASC
```

**Use Case**: Study Pattern Analysis
- Identifies effective reading patterns that correlate with success
- Helps advisors recommend study strategies
- Shows correlation between consistent reading and performance
- Enables early intervention for students with suboptimal reading patterns

**Graph DB Advantage**:
- Efficiently analyzes temporal patterns across multiple relationships
- Easy to correlate reading behavior with performance
- Natural representation of student-textbook-course relationships

### 3. Identify At-Risk Students Based on Textbook Usage

```cypher
// Find students who might be at risk based on textbook interaction patterns
MATCH (student:Student)-[comp:COMPLETED]->(course:Course)
MATCH (course)-[:REQUIRES]->(textbook:Textbook)
OPTIONAL MATCH (student)-[view:VIEWED_PAGE]->(textbook)
WITH student, course, textbook,
     COUNT(view) AS totalViews,
     COLLECT(view.timestamp) AS viewTimes
WITH student, course,
     AVG(totalViews) AS avgViewsPerTextbook,
     CASE WHEN size(viewTimes) > 0
          THEN reduce(s = 0, t in viewTimes |
               s + CASE WHEN size(viewTimes) > 0 
                    THEN 1 ELSE 0 END) / CASE WHEN size(viewTimes) > 0 THEN size(viewTimes) ELSE 1 END
          ELSE 0 END AS examWeekRatio
WHERE (examWeekRatio > 0.7 OR avgViewsPerTextbook < 10) AND avgViewsPerTextbook > 0
RETURN student.id,
       student.name,
       COUNT(DISTINCT course) AS coursesAtRisk,
       ROUND(AVG(avgViewsPerTextbook)) AS avgTextbookViews,
       ROUND(AVG(examWeekRatio) * 100) AS percentageLastMinuteReading
ORDER BY coursesAtRisk DESC, avgTextbookViews ASC
```

**Use Case**: Early Intervention
- Identifies students who might be struggling with study habits
- Detects cramming behavior vs. consistent reading
- Enables proactive academic support
- Helps advisors identify students needing study skills workshops

**Graph DB Advantage**:
- Complex pattern matching across multiple relationships
- Efficient temporal analysis of study behaviors
- Easy correlation of reading patterns with course performance

### 4. Analyze Textbook Effectiveness

```cypher
// Analyze textbook effectiveness by comparing student interactions and performance
MATCH (course:Course)-[:REQUIRES]->(textbook:Textbook)
MATCH (student:Student)-[comp:COMPLETED]->(course)
OPTIONAL MATCH (student)-[view:VIEWED_PAGE]->(textbook)
OPTIONAL MATCH (student)-[interact:INTERACTED_WITH]->(textbook)
WITH course, textbook,
     COUNT(DISTINCT student) AS totalStudents,
     AVG(CASE WHEN comp.grade IN ['A', 'A-', 'B+'] THEN 1 ELSE 0 END) AS successRate,
     AVG(CASE WHEN view IS NOT NULL THEN 1 ELSE 0 END) AS readingRate,
     AVG(CASE WHEN interact.interactionType = 'highlight' THEN 1 ELSE 0 END) AS highlightRate,
     AVG(CASE WHEN interact.interactionType = 'note' THEN 1 ELSE 0 END) AS noteRate
RETURN course.name,
       textbook.name,
       totalStudents,
       ROUND(successRate * 100) AS successPercentage,
       ROUND(readingRate * 100) AS readingPercentage,
       ROUND(highlightRate * 100) AS highlightPercentage,
       ROUND(noteRate * 100) AS notePercentage
ORDER BY successRate DESC
```

**Use Case**: Textbook Evaluation
- Evaluates textbook effectiveness based on student success
- Analyzes different types of engagement (reading, highlighting, notes)
- Helps departments make informed decisions about course materials
- Identifies which textbooks promote active learning

**Graph DB Advantage**:
- Easy aggregation of multiple interaction types
- Efficient correlation of textbook usage with performance
- Natural representation of different interaction relationships

### 5. Find Similar Students Based on Reading Patterns

```cypher
// Find students with similar reading patterns for study group recommendations
// First get a student with textbook views
MATCH (student:Student)-[view1:VIEWED_PAGE]->(textbook:Textbook)
WITH student LIMIT 1  // Get first student with textbook views
MATCH (student)-[view1:VIEWED_PAGE]->(textbook:Textbook)
MATCH (other:Student)-[view2:VIEWED_PAGE]->(textbook)
WHERE other.id <> student.id
WITH student, other, textbook,
     COLLECT(view1.timestamp) AS student1Times,
     COLLECT(view2.timestamp) AS student2Times,
     COUNT(DISTINCT view1) AS student1Views,
     COUNT(DISTINCT view2) AS student2Views
WITH student, other, textbook,
     AVG(ABS(student1Views - student2Views)) AS viewDifference,
     AVG(CASE 
         WHEN size(student1Times) > 0 AND size(student2Times) > 0
         THEN duration.between(
              datetime(replace(student1Times[0], ' ', 'T')),
              datetime(replace(student2Times[0], ' ', 'T'))
         ).hours
         ELSE 24 END) AS timingDifference
WHERE viewDifference < 10 AND timingDifference < 12
WITH other.id AS otherId,
     other.learningStyle AS learningStyle,
     viewDifference,
     timingDifference,
     COLLECT(DISTINCT textbook.name) AS commonTextbooks
RETURN otherId,
       learningStyle,
       ROUND(viewDifference) AS avgViewDifference,
       ROUND(timingDifference) AS avgTimingDifferenceHours,
       commonTextbooks
ORDER BY viewDifference ASC, timingDifference ASC
LIMIT 5
```

### 6. Generate Personalized Textbook Usage Insights

```cypher
// Find textbook consumption patterns of similar but higher-performing students
// Get a sample student with both course completions and textbook interactions
MATCH (target:Student)-[:COMPLETED]->(:Course)
MATCH (target)-[:VIEWED_PAGE]->(:Textbook)
WITH target LIMIT 1  // Use parameter if provided: WHERE target.id = COALESCE($studentId, target.id)
MATCH (target)-[comp:COMPLETED]->(course:Course)
WITH target, course, comp.grade AS grade

// Find similar students based on multiple criteria
MATCH (target)-[styleSim:SIMILAR_LEARNING_STYLE]->(similar:Student)
WHERE styleSim.similarity > 0.7  // High learning style similarity

// Ensure they're pursuing the same degree
MATCH (target)-[:PURSUING]->(degree:Degree)
MATCH (similar)-[:PURSUING]->(degree)

// Find students who performed better in the same courses
MATCH (similar)-[simComp:COMPLETED]->(course)
WHERE simComp.grade > grade  // Better grade in same course

// Get their textbook interactions for this course
MATCH (similar)-[view:VIEWED_PAGE]->(textbook:Textbook)
WHERE view.courseId = course.id

// Calculate reading patterns
WITH target, similar, course, textbook, view,
     duration.between(
       datetime(replace(view.timestamp, ' ', 'T')),
       datetime(replace(view.timestamp, ' ', 'T')) + duration({minutes: view.duration})
     ) AS readingSession

// Group by student and course to analyze patterns
WITH target, similar, course,
     COUNT(DISTINCT view) AS totalPageViews,
     COUNT(DISTINCT date(datetime(replace(view.timestamp, ' ', 'T')))) AS uniqueReadingDays,
     AVG(view.duration) AS avgReadingDuration,
     MIN(view.timestamp) AS firstRead,
     MAX(view.timestamp) AS lastRead,
     duration.between(
       datetime(replace(MIN(view.timestamp), ' ', 'T')),
       datetime(replace(MAX(view.timestamp), ' ', 'T'))
     ) AS readingSpan

// Calculate reading consistency and intensity
WITH target, similar, course,
     totalPageViews,
     uniqueReadingDays,
     avgReadingDuration,
     readingSpan,
     CASE 
       WHEN readingSpan.days > 0 
       THEN toFloat(uniqueReadingDays) / readingSpan.days 
       ELSE 0 
     END AS readingConsistency,
     CASE 
       WHEN readingSpan.days > 0 
       THEN toFloat(totalPageViews) / readingSpan.days 
       ELSE 0 
     END AS readingIntensity

// Get target student's patterns for comparison
MATCH (target)-[targetView:VIEWED_PAGE]->(targetTextbook:Textbook)
WHERE targetView.courseId = course.id
WITH target, similar, course,
     totalPageViews,
     uniqueReadingDays,
     avgReadingDuration,
     readingConsistency,
     readingIntensity,
     COUNT(DISTINCT targetView) AS targetPageViews,
     COUNT(DISTINCT date(datetime(replace(targetView.timestamp, ' ', 'T')))) AS targetReadingDays,
     AVG(targetView.duration) AS targetAvgDuration

// Calculate differences in reading patterns
WITH target, similar, course,
     totalPageViews - targetPageViews AS pageViewDiff,
     uniqueReadingDays - targetReadingDays AS readingDaysDiff,
     avgReadingDuration - targetAvgDuration AS durationDiff,
     readingConsistency,
     readingIntensity

// Aggregate insights across all similar students
WITH target, course,
     AVG(pageViewDiff) AS avgPageViewDiff,
     AVG(readingDaysDiff) AS avgReadingDaysDiff,
     AVG(durationDiff) AS avgDurationDiff,
     AVG(readingConsistency) AS avgReadingConsistency,
     AVG(readingIntensity) AS avgReadingIntensity,
     COUNT(DISTINCT similar) AS similarStudentCount

// Format recommendations
RETURN course.id AS courseId,
       course.name AS courseName,
       similarStudentCount AS numberOfSimilarStudents,
       CASE 
         WHEN avgPageViewDiff > 0 THEN 'Read more pages'
         ELSE 'Read fewer pages but more thoroughly'
       END AS pageViewRecommendation,
       CASE 
         WHEN avgReadingDaysDiff > 0 THEN 'Spread reading across more days'
         ELSE 'Consolidate reading into fewer, longer sessions'
       END AS readingScheduleRecommendation,
       CASE 
         WHEN avgDurationDiff > 0 THEN 'Spend more time per reading session'
         ELSE 'Take shorter, more focused reading breaks'
       END AS durationRecommendation,
       CASE 
         WHEN avgReadingConsistency > 0.5 THEN 'Maintain regular reading schedule'
         ELSE 'Establish more consistent reading habits'
       END AS consistencyRecommendation,
       CASE 
         WHEN avgReadingIntensity > 0.5 THEN 'Increase overall reading intensity'
         ELSE 'Focus on quality over quantity'
       END AS intensityRecommendation,
       avgPageViewDiff,
       avgReadingDaysDiff,
       avgDurationDiff,
       avgReadingConsistency,
       avgReadingIntensity
ORDER BY similarStudentCount DESC, avgPageViewDiff DESC
LIMIT 10
```

**Use Case**: Academic Advising
- Provides personalized textbook usage recommendations based on successful students
- Analyzes reading patterns across multiple dimensions (frequency, duration, consistency)
- Helps students optimize their study habits by comparing with similar but higher-performing peers
- Generates actionable insights for improving textbook engagement

**Graph DB Advantage**:
- Efficiently finds similar students based on learning style and degree program
- Natural representation of temporal reading patterns and relationships
- Easy comparison of reading behaviors between target and similar students
- Complex pattern matching across multiple relationship types (learning style, course completion, textbook usage)

**Key Metrics**:
- Page View Differences: Compares total pages read between target and similar students
- Reading Days: Analyzes how reading is distributed across time
- Session Duration: Examines typical reading session lengths
- Reading Consistency: Measures regularity of reading habits
- Reading Intensity: Evaluates overall reading engagement

**Returned Columns**:
- courseId/courseName: Identifies the course being analyzed
- numberOfSimilarStudents: Number of similar students used for comparison
- Various Recommendations: Personalized advice for improving reading habits
- Difference Metrics: Detailed comparisons with similar students' patterns

## Common Queries for Hackathon

### Find Sample IDs to Test Queries

**Important**: All example queries now use dynamic ID selection, but you can also use specific IDs:
```cypher
// Get a random student who has completed courses and is pursuing a degree
MATCH (s:Student)-[:PURSUING]->(d:Degree)
WHERE EXISTS((s)-[:COMPLETED]->(:Course))
RETURN s.id, s.name, d.name, COUNT{(s)-[:COMPLETED]->(:Course)} as coursesCompleted
LIMIT 10
```

### Find Available Terms
```cypher
// List all terms in the database
MATCH (t:Term)
RETURN t.id, t.name, t.startDate, t.endDate
ORDER BY t.startDate DESC
```

### Explore Course Prerequisites
```cypher
// Show prerequisite chains for Computer Science courses
MATCH path = (prereq:Course)-[:PREREQUISITE_FOR*1..3]->(course:Course)
WHERE course.department = 'Computer Science'
RETURN path
LIMIT 25
```

### Student Performance Analysis
```cypher
// Find high-performing students to learn from
MATCH (s:Student)-[c:COMPLETED]->(course:Course)
WHERE c.grade IN ['A', 'A-']
WITH s, COUNT(DISTINCT course) as excellentCourses
WHERE excellentCourses > 5
RETURN s.id, s.name, s.learningStyle, excellentCourses
ORDER BY excellentCourses DESC
LIMIT 10
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Import Errors

**Problem**: "Couldn't load the external resource"
**Solution**: 
- Ensure Neo4j has started completely before running queries
- Check that APOC plugin is installed and enabled
- For CSV import, verify files are in the correct import directory

**Problem**: "Constraint already exists"
**Solution**:
- This is fine! The IF NOT EXISTS clause should handle it
- If it still errors, you can skip constraint creation if they already exist

#### 2. Memory Issues

**Problem**: "Java heap space" or slow queries
**Solution**:
```bash
# Increase Neo4j heap memory in neo4j.conf:
dbms.memory.heap.initial_size=2G
dbms.memory.heap.max_size=4G
dbms.memory.pagecache.size=2G
```

#### 3. Python Generator Issues

**Problem**: "ModuleNotFoundError: No module named 'faker'"
**Solution**:
```bash
# Make sure you're in the virtual environment
source venv/bin/activate  # macOS/Linux
venv\Scripts\activate     # Windows

# Reinstall requirements
pip install -r requirements.txt
```

**Problem**: Generator runs but no output
**Solution**:
- Check that you have write permissions in the directory
- Look for the `umbc_data/` folder after running
- Check console for any error messages

#### 4. Query Performance

**Problem**: Queries are running slowly
**Solution**:
1. Ensure indexes are created (see Step 3)
2. Use EXPLAIN or PROFILE to analyze query:
```cypher
PROFILE MATCH (s:Student)-[:COMPLETED]->(c:Course)
RETURN COUNT(*)
```
3. Add more specific WHERE clauses to limit data early
4. Consider using LIMIT during testing

#### 5. Timestamp Issues in Queries

**Note**: The generated timestamps are in "YYYY-MM-DD HH:MM:SS" format. When working with temporal queries, use:
```cypher
// Convert string timestamp to datetime
datetime(replace(timestamp_string, ' ', 'T'))

// Example:
WITH '2023-09-15 14:30:00' AS ts
RETURN datetime(replace(ts, ' ', 'T')) AS converted_datetime
```

### Getting Help

1. **Neo4j Documentation**: https://neo4j.com/docs/
2. **APOC Documentation**: https://neo4j.com/docs/apoc/
3. **Graph Data Science Docs**: https://neo4j.com/docs/graph-data-science/
4. **Cypher Reference**: https://neo4j.com/docs/cypher-manual/

### Dataset & Query Support

For issues with the UMBC dataset, data import problems, or questions about example queries:
- **Email**: Jason Paluck - paluck@umbc.edu

### Quick Debug Queries

```cypher
// Check if data loaded correctly
CALL db.labels() YIELD label
RETURN label;

// Check relationship types
CALL db.relationshipTypes() YIELD relationshipType
RETURN relationshipType;

// See sample of each node type
MATCH (s:Student) RETURN s LIMIT 1
UNION
MATCH (c:Course) RETURN c LIMIT 1
UNION
MATCH (f:Faculty) RETURN f LIMIT 1
UNION
MATCH (d:Degree) RETURN d LIMIT 1;

// Check database statistics
CALL db.stats.retrieve("GRAPH COUNTS")
YIELD data
RETURN data;
```

## Tips for Hackathon Success

1. **Start Simple**: Begin with basic queries before attempting complex ones
2. **Use the Browser Guide**: The included HTML guide has tested queries
3. **Leverage Indexes**: Always create indexes before importing large datasets
4. **Test with LIMIT**: Add LIMIT to queries during development
5. **Explore Relationships**: Use the visualization to understand connections
6. **Check Node Properties**: Use `MATCH (n:NodeType) RETURN keys(n) LIMIT 1` to see available properties

## Dataset Statistics

After successful import, you should have:
- **500 Students** with diverse learning styles and preferences
- **100 Courses** across Computer Science and Biology
- **30 Faculty** members distributed across departments
- **4 Degree Programs** (BS/BA in CS and Biology)
- **12 Terms** of academic history
- **~80-100 Textbooks** with reading patterns
- **~10,000 Course Completions** with grades
- **~15,000 Textbook Interactions** (views and interactions)
- **Multiple relationship types** for similarity, prerequisites, and requirements

Happy Hacking!
