# User Guide for Lecturer
## Introduction:
In this section we explain how a lecturer can use the application.

In the Dashboard, lecturers have the capability to create new courses or modify existing ones. Within each specific course, instructors can also add corresponding chapters, sections, and various types of content such as media, quizzes, and flashcards during the course creation process.
## Add Course
Via the 'Add course' button on the dashboard the lecturer can create a new course by entering following details:
![create Course](Images/create_course.png)
1. In course details, lecturer has to enter Title, description and number of chapters as they want.
2. Start Date, end date and semester.
3. And by clicking on 'Confirm' the course is created successfully after clicking on "Create Course" button. Lecturer can also publish course or by disabling this option he or she just creates the course without publishing.
   ![Publish course](Images/Publish_course.png)
## Add Chapter details
![Adding Chapter](Images/Adding_Chapter.png)
1. After adding a course the lecturer can create new chapters and modify existing ones. For example, he can set start and end dates along with other settings.
2. Lecturer Can enter the details of chapters like adding section and contents of particular chapter.
3. Lecturer can also modify course details from this page.
   ![Course modification](Images/Course_modification.png)

## Add section and Content
1. Lecturer can add a section via "ADD SECTION" button.
   ![Add section](Images/Add_section.png)
2. After adding a stage via "ADD STAGE" to a section, lecturer can create content via "ADD CONTENT".
3. With that option lecturer can add various types of content like Quiz,Flashcard or Media.

   3.1  ADD FLASHCARD
    - Before creating the flashcard, the lecturer has to fill out the required details for the flashcard, as seen here.
      ![add flashcard](Images/add_flashcard.png)
    - After clicking on "SAVE" flashcard is created successfully.

   3.2  ADD MEDIA
    - By adding Content details like name, date reward point and tags(optionally).
    - By adding media via "ADD MEDIA", lecturer can select media of any type like images, pdfs and videos etc.
    ![add media](Images/add_media.png)
    - And on clicking "ADD FILES" lecturer can upload files.
      ![upload media](Images/upload_media.png)


   3.3  ADD QUIZ
   - Provide the quiz's information related to the content.
     ![add quiz](Images/add_quiz.png)

   (a-code-assignment)=
   3.4 ADD CODE ASSIGNMENT
   - Ensure that in your Github Organization, which will be used to store students repositories, **exactly one** Github Classroom exists with the same name as the MEITREX course.
   - Create the assignment on the Github Classroom page:
     - Provide a starter repository.
     - Add a `README.md` file to the starter repository.
     - Set student repository visibility to public (there's a bug on the Github side so it MUST NOT be private as of now)
     - Add `yaml` autograding configuration as follows:
```yaml
name: Autograding Tests
'on':
  - push
  - repository_dispatch
permissions:
  checks: write
  actions: read
  contents: read
        
jobs:
  run-autograding-tests:
  runs-on: ubuntu-latest
  if: github.actor != 'github-classroom[bot]'
  steps:
    - name: Checkout code
      uses: actions/checkout@v4
        
    # Add further tests as specified below, and extend "env" and "with" sections with the added tests.
    # "command" can be anything that runs tests, it doesn't matter if it is Gradle or any other tool. 
    # ⚠️ Note: For tools like npm, you may need to install dependencies first.
    # Example for JS:
    # - name: Install NPM deps
    #   run: npm install

    - name: Deposit increases balance
      id: deposit-increases-balance
      uses: classroom-resources/autograding-command-grader@v1
      with:
        test-name: BankTest - Deposit increases balance
        command: 'gradle test --tests "bank.BankTest.testDepositIncreasesBalance"'
        timeout: 1
        max-score: 5
        
    - name: Add to cart and place order
      id: shop-add-to-cart
      uses: classroom-resources/autograding-command-grader@v1
      with:
        test-name: OnlineShopTest - Add to cart and place order
        command: 'gradle test --tests "shop.OnlineShopTest.testAddToCartAndPlaceOrder"'
        timeout: 2
        max-score: 5

    # Template for additional tests:
    - name: <TestName> # will be shown in the students' feedback summary 
      id: <test-id>
      uses: classroom-resources/autograding-command-grader@v1
      with:
        test-name: <TestGroupName> - <TestName>
        command: '<command to run the test>'
        timeout: <how long to wait for the test to complete in minutes>
        max-score: <max score for this test>
 
    # The test name must follow the format: `<TestGroupName> - <TestMethodName>`.
    # Tests with the same group name will be grouped together in the feedback shown to students.

    - name: Autograding Reporter
      uses: classroom-resources/autograding-grading-reporter@v1
      env:
        DEPOSIT-INCREASES-BALANCE_RESULTS: "${{steps.deposit-increases-balance.outputs.result}}"
        SHOP-ADD-TO-CART_RESULTS: "${{steps.shop-add-to-cart.outputs.result}}"
        # for each additional test, add a new environment variable
        # TEST-ID_RESULTS: "${{steps.test-id.outputs.result}}"
      with:
        # Add new test ids that must be executed. 
        # They must be split only by a comma, nothing else in between (not even spaces)
        runners: deposit-increases-balance,shop-add-to-cart
 ```
   - Feedback example based on a correct `yaml` configuration:
     ![code feedback example](Images/code_feedback.png)

   - Each failed test's output from the Github Actions run, which is used for autograding, will appear in the dropdown of the corresponding test in MEITREX. Use such configuration that provides meaningful feedback to students.
     E.g. for `Gradle KTS` the following configuration will result in a output similar to the one shown below:

```kotlin
    tasks.withType<Test>().configureEach {
        testLogging {
            events("passed", "skipped", "failed")
            showExceptions = true
            showCauses = true
            showStackTraces = true
            exceptionFormat = org.gradle.api.tasks.testing.logging.TestExceptionFormat.FULL
            showStandardStreams = true
        }
    }
``` 

![not passed code feedback example](Images/not_passed_test_feedback.png)

   - After creating the assignment on Github Classrooom, go to MEITREX
   - Click on "ADD CODE ASSIGNMENT" in the course content page.
   - Sync the assignments from Github Classroom by clicking on "Sync Assignments" button.
     ![sync code assignments](Images/sync_code_assignments.png)
   - Select the assignment you want to add to the course.
     ![synced code assignments](Images/synced_code_assignments.png)
   - Provide the code assignment's information related to the content.
     ![add code assignment](Images/add_code_assignment.png)

## Add contents details (MEDIA, FLASHCARD, QUIZ)

After adding flashcards, media and quiz lecturer can further add details in flashcard, quiz or media what he created.

### Flashcard Details
1. Add flashcard by clicking on "ADD FLASHCARD".
2. Add the item details.![quiz detail](Images/Item_Information.png)
3. Add Sides of flashcard.As a side lecturer can create question and answer of that particular Flashcard.
   ![add que-ans](Images/add_que-ans.png)
4. Flashcards and their sides are deletable.

### Media Details
1. After adding media, lecturer can see that uploaded content by clicking on particular uploaded content and he or she can see that content, even delete and edit option is also there.

### Quiz Details
1. Add question to the quiz by "ADD QUESTION".
2. Add the item details
3. For the question type, 3 options are given:
   ![quiz detail](Images/quiz_detail.png)

- **Multiple Choice Question:** :
  Students have to select all the correct answers from the available options.
    - Fill the required details like question, hint etc.
    - By clicking "Add", multiple choice question will be added.
      ![add asosiaction](Images/multiple_choice.png)
- **Cloze Question:**
  Fill in missing words in a sentence.
    - lecturer can add the cloze question by adding required details shown in the Image.
      ![add cloze](Images/Cloze_Question.png)
- **Association Question:**
  Match related concepts or items.
    - In assosication question lecturers can create question-answer pairs for assessments.
      ![add asosiaction](Images/Association_Quiz.png)
## Join as Lecturer
After creating the course the lecturer is automatically added as the administrator to the course.

## Upgrade members in a course
An administrator of the course can set the roles of all users of a course on the members site. The members sites is accessible through the course page, where you can find the button in the top right corner, where also the settings of the course and the "Add Chapter" button are.
The administrator can set the users to be students, tutors or administrators.





