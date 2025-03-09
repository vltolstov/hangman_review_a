https://github.com/LlqWst/HangMan_Java.git    
[Ivan]

Игра в процедурном стиле, выполнена в одном классе.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

Команды не работают
```
|------------------------------Виселица-----------------------------|
|Для начала игры введите 'старт', для выхода из игры введите 'выход'|
|Ввод должн быть без одинарных кавычек                              |
|-------------------------------------------------------------------|
Старт
Уведомление: Некорректный ввод!
Ожидается 'старт' или 'выход'
Ввод должн быть без одинарных кавычек
```

## ХОРОШО

+ 👍 Игра запускается
+ 👍 Можно ввести только одиночную букву русского языка
+ 👍 Есть список введенных букв

## ЗАМЕЧАНИЯ

**1. Нейминг**

- В названии переменных не пиши тип данных, к которым они относится. И вообще не употребляй венгерскую ноттацию
```
List<Character> incorrectCharsList = new ArrayList<>();
List<String> listWithWords = readWordsFromFile();

//ПРАВИЛЬНО:
List<Character> incorrectChars = new ArrayList<>();
List<String> words = readWordsFromFile();
```

- Не экономь буквы. Если словосочетание не слишком большое, а оно тут не слишком большое, то пиши слова полностью.
К тому же здесь избыточный контекст
```
int getRandInt(int linesNumber)

//ПРАВИЛЬНО:
int getRandom(int linesNumber)
```

- С большой буквы пишутся только названия классов
```
String PlayerInput

//ПРАВИЛЬНО:
String playerInput
```

- Название метода и название входящих в него аргументов должны как можно лучше описывать то, что делает метод
```
private static boolean isInputCorrect(String PlayerInput) {
  return PlayerInput.length() == 1 && !isSymbolNotCyrillic(PlayerInput.charAt(0));
}

//ЛУЧШЕ:
private static boolean isOneLetter(String s) {
  return s.length() == 1 && !isSymbolNotCyrillic(s.charAt(0));
}
```

- Избыточно
```
private static char getLowCaseChar(String playerInput)

//ЛУЧШЕ:
private static char toLowerCase(String playerInput)
```

- Название метода вводит в заблуждение. "Инициализация" означает одномоментное действие по установке значений, а в этом методе в цикле крутится основная игровая логика
```
private static void gameInit() {
  //...
  do {
    //...
  } while (mistakeCount < MAX_MISTAKES && !seekWord.equals(maskWord));
  //...
}

//ПРАИЛЬНО:
private static void start() {...}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Нарушение DRY**, магические буквы, числа, слова. Вводи константы
```
System.out.println("|Для начала игры введите 'старт', для выхода из игры введите 'выход'|");
case "старт": //...;
case "выход": //...; 
System.out.println("Ожидается 'старт' или 'выход'");

//ПРАВИЛЬНО:
private final static String CMD_START = "старт";
private final static String CMD_EXIT = "выход";

System.out.printf("|Для начала игры введите '%s', для выхода из игры введите '%s'| %n", CMD_START, CMD_EXIT);
case CMD_START: //...;
case CMD_EXIT: //...; 
System.out.printf("Ожидается '%s' или '%s' %n", CMD_START, CMD_EXIT);
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*   
*refactoring.guru "Замена магического числа символьной константой"*  

**3. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками. Исключение- метод equals()
```
if (awaitInitialInput() == EXIT) System.exit(0);

//ПРАВИЛЬНО:
if (awaitInitialInput() == EXIT) {
  System.exit(0);
}    
```

**4. Сразу возвращай результат**, не пиши избыточный код
```
private static int awaitInitialInput() {
  int entry = 0;
  do {
    switch (getPlayerInput()) {
      case "старт":
        entry = START;
        break;
      case "выход":
        entry = EXIT;
        break;
        //...
    }
  } while (entry != START && entry != EXIT);
  return entry;
}

//ПРАВИЛЬНО:
private static int awaitInitialInput() {
  while(true) {
    switch (getPlayerInput()) {
      case "старт":
        return START;
      case "выход":
        return EXIT;
        //...
    }
  } 
}
```

**5. Избыточно**, сделай `scanner` полем класса, а не метода- тогда он создастся один раз. Сейчас этот объект пересоздается каждый раз при вызове метода
```
public class Main {
  private static String getPlayerInput() {
    Scanner scanner = new Scanner(System.in);
    return scanner.nextLine();
  }
}

//ПРАВИЛЬНО:
public class Main {
  private static final Scanner SCANNER = new Scanner(System.in);

  private static String getPlayerInput() {   
    return SCANNER.nextLine();
  }
}
```

**6. Излишнее дробление**
```
private static String getRandomWord(List<String> listWithWords) {
  int numberLine = getRandInt(listWithWords.size());
  return listWithWords.get(numberLine);
}

private static int getRandInt(int linesNumber) {
  Random rand = new Random();
  return rand.nextInt(linesNumber);
}

//ПРАВИЛЬНО:
private static final Random RANDOM = new Random();

private static String getRandomWord(List<String> listWithWords) {
  int size = listWithWords.size();  
  int numberLine = RANDOM.nextInt(size);
  return listWithWords.get(numberLine);
}
```

**7. Положительные условия читаются лучше отрицательных**
```
private static boolean isSymbolNotCyrillic(char inputChar) {
  return Character.UnicodeBlock.of(inputChar) != Character.UnicodeBlock.CYRILLIC;
}

//ЛУЧШЕ:
private static boolean isCyrillic(char c) {
  return Character.UnicodeBlock.of(inputChar) == Character.UnicodeBlock.CYRILLIC;
}
```

И тогда станет понятнее эта запись
```
//БЫЛО:
private static boolean isInputCorrect(String PlayerInput) {
  return PlayerInput.length() == 1 && !isSymbolNotCyrillic(PlayerInput.charAt(0));
}

//СТАНЕТ:
private static boolean isInputCorrect(String PlayerInput) {
  return PlayerInput.length() == 1 && isCyrillic(PlayerInput.charAt(0));
}
```

**8. Избыточно**, слово `maskWord` содержит 8 букв, а `return` - только 6. Поэтому `maskWord` здесь не подходит на роль поясняющей переменной. Еще лучше `tempString`(дважды неудачное название) переименовать на `result`
```
StringBuilder tempString = new StringBuilder(maskWord);
//...
maskWord = tempString.toString();
return maskWord;

//ПРАВИЛЬНО:
StringBuilder result = new StringBuilder(maskWord);
//...
return result.toString();
```

**9. Создавай вспомогательные методы или поясняющие переменные**, делай программу более простой и понятной
```
do {...} while (mistakeCount < MAX_MISTAKES && !seekWord.equals(maskWord)); 

//ПРАВИЛЬНО:
do {
  //...
  boolean isGameOver = mistakeCount >= MAX_MISTAKES || seekWord.equals(maskWord);  
} while (!isGameOver); 
```

**10. Картинки висельницы вынеси в константы**

**11. Еще раз про нейминг**. Точно-точно названия здесь должны содержать по 5 слов? Это тяжело даже прочитать, придумывай лаконичные, но понятные названия
```
if (isDuplicatedIncorrectChar(inputChar, incorrectCharsList)) {
  printNotificationDuplicatedIncorrectChar();
} else if (isDuplicatedCorrectChar(inputChar, maskWord)) {
  printNotificationDuplicatedCorrectChar();
} else if (isWordContainsInputChar(seekWord, inputChar)) {
  maskWord = rebuildMaskWord(seekWord, inputChar, maskWord);
}
```

## ВЫВОД

Для процедурного стиля приемлемо.

n.73(138)  
#ревью #виселица