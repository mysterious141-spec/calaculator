# Stacks in the Real World: Undo/Redo Editor

Assignment: describe real-world applications that need a **stack**, explain why a stack is necessary, and develop one application.

---

## 1. Real-world applications of stacks

| Application | How a stack is used |
|---|---|
| Undo/redo in editors | Each edit is pushed. Undo pops the latest edit and pushes it onto a redo stack. |
| Browser back/forward | Visited pages are pushed. Back pops the current page. |
| Function call stack | Each call pushes a frame with its local state. A return pops it. |
| Bracket and syntax checking | Opening brackets are pushed. A closing bracket must match the top. |
| Expression evaluation | Calculators and compilers convert infix to postfix and evaluate it with a stack. |
| DFS and backtracking | Maze, Sudoku and route solvers push choices and pop to go back. |
| Transaction rollback | Steps are reversed in the opposite order they were applied. |
| HTML/XML tag matching | Browsers push opening tags and pop on the matching closing tag. |

## 2. Why stacks are necessary

All of these problems need **LIFO** (last in, first out) behavior: the most recent item must be handled first.

- **Order is enforced.** You cannot undo step 2 before step 5, because only the top of the stack is reachable.
- **Nested work fits naturally.** Nested function calls, nested brackets and nested tags all close in reverse order of opening.
- **It is fast.** Push, pop and peek are all O(1).
- **Alternatives need extra work.** A queue or a plain list would need extra bookkeeping and invite ordering bugs.

## 3. Chosen application: Undo/Redo text editor

Two stacks do the work:

- **Undo stack:** every edit is executed, then pushed here.
- **Redo stack:** undone edits wait here until redone.
- A **new edit clears the redo stack**, because that future no longer exists.

```java
private void run(Command c) {
    c.execute();
    undoStack.push(c);
    if (undoStack.size() > HISTORY_LIMIT) undoStack.removeLast(); // drop oldest
    redoStack.clear();                                            // new edit invalidates redo
}

public boolean undo() {
    if (undoStack.isEmpty()) return false;
    Command c = undoStack.pop();
    c.undo();
    redoStack.push(c);
    return true;
}
```

**Design:** the command pattern. Each action (`InsertCommand`, `DeleteCommand`) stores only the change and knows how to run and reverse itself. This uses far less memory than saving a full copy of the text on every edit, and a new action only needs a new command class.

**Features**
- Cursor-based insert and backspace (edit anywhere in the text)
- Undo/redo restores the cursor position
- History limit of 100 edits
- 7 unit tests, run automatically by GitHub Actions on every push
- A browser version that shows both stacks changing live

## 4. Run it

**Java version**
```bash
mvn test
mvn compile
java -cp target/classes com.alex.editor.Main
```
Commands: `type <text>`, `delete <n>`, `left [n]`, `right [n]`, `home`, `end`, `undo`, `redo`, `exit`

**Web version:** open `docs/index.html` in a browser.

## 5. Project structure

```
undo-redo-editor/
├── .github/workflows/ci.yml        Runs mvn test on every push
├── docs/
│   └── index.html                  Web version of the editor
├── src/main/java/com/alex/editor/  Command, Document, InsertCommand, DeleteCommand, Editor, Main
├── src/test/java/com/alex/editor/  EditorTest
├── pom.xml
└── README.md
```

## 6. Possible improvements
- Merge consecutive typing into a single undo step
- Text selection and copy/paste commands
- Save and load files
