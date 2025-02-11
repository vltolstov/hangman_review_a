https://github.com/PetrovYegor/Hangman  
[Егор Петров]

Игра в процедурном стиле, состоит из двух классов. Все работает как надо.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Не обнаружено

## ХОРОШО

1. Игра запускается
2. Есть список введенных букв
3. Команды не чувствительны к регистру
4. Можно ввести только одиночную букву русского алфавита
5. Сообщает о повторном вводе отсутствующей в слове буквы
6. Широко применяются константы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- (±)Если придерживаться Oracle Java code conventions, эту константу нужно писать стилем UPPER_SNAKE.  
А если придерживаться конвенции Гугл, то так, как здесь написано
```
private static final Map<Character, Integer> lettersOfGuessingWord = new HashMap<>()
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  

**2. Создавай вспомогательные методы**, делай программу более простой и понятной
```
while (wrongAnswersCounter <= WRONG_ANSWERS_LIMIT) {...}

//ПРАВИЛЬНО:
while (!isLose()) {...}

private static boolean isLose() {
  return wrongAnswersCounter >= WRONG_ANSWERS_LIMIT;
}
```

**3. Чтение файла тоже следует вынести в отдельный класс**, раз проект все равно уже состоит не из одного класса.

**4. Избыточно** и сложно, нет необходимости в методе, который будет возвращать состояния игры в виде строковых значений
```
private static String getResultOfRound(int wordLength) {
  if (wrongAnswersCounter == WRONG_ANSWERS_LIMIT) {
    return PLAYER_LOSE;
  } else if (solvedLettersCounter == wordLength) {
    return PLAYER_WIN;
  } else {
    return GAME_NOT_FINISHED;
  }
}
```

Это игра, нужно создать методы, которые без посредников определят выигрыш или проигрыш
```
if (isWin()) {
  printWinMessage();  
} else if (isLose()) {
  printLoseMessage(); 
}
```

**5. Какая-то сложная неочевидная логика:** секретное слово преобразуется в хэшмап, где ключ это буква, а значение это цифра. Цифра означает количество таких букв в слове
```
private static void initializeLettersOfGuessingWord(String word) {
  for (int i = 0; i < word.length(); i++) {
    Character currentChar = word.charAt(i);
    if (!lettersOfGuessingWord.containsKey(currentChar)) {
      lettersOfGuessingWord.put(currentChar, 1);
    } else {
      lettersOfGuessingWord.put(currentChar, lettersOfGuessingWord.get(currentChar) + 1);
    }
  }
}
```

Далее при угадывании буквы, из мапы по ключу этой буквы берется число, которое плюсуется к счетчику отгаданных букв. А потом в мапе это число обнуляется
```
if (lettersOfGuessingWord.containsKey(guess)) {
  solvedLetters.add(guess);
  solvedLettersCounter += matchesCount(guess);
  lettersOfGuessingWord.put(guess, 0);
}
```

И когда счетчик отгаданных букв будет равен длине секретного слова, значит выиграли
```
if (solvedLettersCounter == wordLength) {
  return PLAYER_WIN;
}
```

Это конечно работает, но разобраться в этом сложно- неочевидно и избыточно.
Намного лучше, потому что проще и понятней, стандартные решения. Например, такое
```
1. При инициализации создать маску: String "ветер" --> String "*****"
2. При отгадывании буквы открывать букву в маске: 'е' --> "*е*е*"
3. Если секретное слово и маска равны, значит выиграли: секретное слово "ветер" эквивалентно маске "ветер" --> отгадали слово
```

**6. Избыточно**, хотя и дискуссионно. Мне представляется, что регулярные выражения здесь- оверинжиниринг
```
private static final Pattern russianAlphabet = Pattern.compile("[а-яёА-ЯЁ]");

private static boolean isRussianAlphabetLetter(char source) {
  return String.valueOf(source).matches(russianAlphabet.toString());
}

//ЛУЧШЕ:
private static boolean isRussianAlphabetLetter(char c) {
  c = Character.toLowerCase(c);
  return c=='ё' || c >= 'а' && c <= 'я';
}
```

**7. class GallowsDrawing**

+ (+)Отдельный класс с картинками хангмана и методом их распечатки, это хорошо.

- Хранить картинки в енаме, а потом грузить их в хешмап- избыточно. Достаточно хранить картинки в массиве, а доступ к картинкам осуществлять по индексу элемента в массиве. 
В принципе енам + хешмап в твоей реализации выполняют функцию простого массива
```
private static final Map<Integer, GallowsStates> numberedGallowStates;

static {
  numberedGallowStates = new HashMap<>();
  numberedGallowStates.put(0, GallowsStates.DEFAULT);
  //еще миллион строк
}

private enum GallowsStates {
  DEFAULT("""
    _____________
        | \\_______  |
        |        \\| |
                  | |
                  | |
                  | |
                  | |
                  | |
     _____________|_|
        """),
    //oths
}        

public static void drawCurrentGallowsState(int stateNumber) {
  GallowsStates state = numberedGallowStates.get(stateNumber);
  if (state == null) {
    throw new IllegalArgumentException(ILLEGAL_ARGUMENT_MESSAGE + stateNumber);
  }
  System.out.println(state.getState());
}

//ПРАВИЛЬНО:
private static final String[] PICTURES = {
    """
    -----   
    |       
    |       
    |       
    |       
    |       
    ------- 
    """,
   """
     -----   
     |   |   
     |   O   
     |       
     |       
     |       
     ------- 
     """
   ,
   // more pics
};

void printPicture(int numPicture) {  
   System.out.println(PICTURES[numPicture]);
}
```

## ВЫВОД

Излишнее увлечение HashMap'ами, из-за этого код местами трудно читается. HashMap'ы не должны быть золотым молотком.  В остальном особых претензий нет.

n.068(125)  
#ревью #виселица    