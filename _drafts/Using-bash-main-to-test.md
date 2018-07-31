https://stackoverflow.com/questions/27472540/difference-between-and-in-bash

for FILE in `find . -name *TestCase.java`;do mkdir -p $(dirname "${FILE/main/test}"); mv $FILE "${FILE/main/test}"; done