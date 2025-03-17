https://github.com/vltolstov/Hangman-OOP  
[Владимир]

Игра в ООП стиле.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Одна и та же буква в разных регистрах считается двумя разными буквами и увеличивает счетчик ошибок
```
ы
Такой буквы в слове нет. Осталось попыток: 2
пр---
Ы
Такой буквы в слове нет. Осталось попыток: 1
```
2. Вместо одной буквы можно ввести целое слово и игра это примет
3. Нет списка неправильно введенных букв

## ХОРОШО

+ 👍 Игра запускается
+ 👍 На старте есть подсказка- открываются две случайные буквы 
+ 👍 Принимает буквы только русского алфавита

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Однобуквенные переменные могут использоваться только в определенных ситуациях: в циклах, лямбдах и т.д. Но здесь- нет
```
int a = 1;
boolean s = status.getGameStatus();
```
Эти переменные, правда, вообще в программе не задействованы. Но тогда их просто нужно убрать.

- Если в проекте есть класс Dictionary, то все переменные с именем, включающим это название, должны быть экземплярами этого класса
```
public class Dictionary {
  private List<String> dictionary;
  //...
}  

//ПРАВИЛЬНО:
public class Dictionary {
  private List<String> words;
  //...
}  
```

- Название должно как можно лучше объяснять суть явления
```
GameField.setRandomOpenLetters()
GameField.field
GameField.resetField()

//ЛУЧШЕ:
GameField.openRandomLetters()
GameField.mask
GameField.createMask()
```

- Название должно объяснять, что делает класс. `class Loop` объясняет только то, что в нем что-то выполняется в цикле. Я бы его назвал `GameLoop`.

- Добавляется один символ
```
public void addDuplicateWrongChars(Character letter)

//ПРАВИЛЬНО:
public void addDuplicateWrongLetter(Character letter)
```

- Геттеры должны что-то возвращать. Если метод void, он не геттер
```
public void getCongratulations() {
  System.out.println("Победа!");
}

//ПРАВИЛЬНО:
void showCongratulations()
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Нарушение DRY**, магические буквы, числа, слова. Вводи константы 
```
System.out.println("Для старта новой игры введите 'Старт'. Для выхода из игры введите 'Стоп'");

if (commandFromUserInput.equals("старт")) {...}
if (commandFromUserInput.equals("стоп")) {...}

//ПРАВИЛЬНО:
private final static String CMD_START = "Старт";
private final static String CMD_STOP = "Стоп";

System.out.printf("Для старта новой игры введите '%s'. Для выхода из игры введите '%s' %n", CMD_START, CMD_STOP);

if (commandFromUserInput.equals(CMD_START)) {...}
if (commandFromUserInput.equals(CMD_STOP)) {...}
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*   
*refactoring.guru "Замена магического числа символьной константой"*  

**3. Нарушение DRY**, магические буквы, числа, слова.  
Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов, либо перенеси ее в третий класс
```
class Result {
  //...
  if (!gameField.getField().contains("-") && mistake.getMistakesCount() != 0) {...}
}

class Status {
  //
  if (!gameField.getField().contains("-") || mistake.getMistakesCount() == 0) {...}
}

class GameField {
  //...
  for (int i = 0; i < this.secretWord.length(); i++) {
    this.field += "-"; 
  }
}

//ПРАВИЛЬНО:
class Result {
  //...
  if (!gameField.getField().contains(GameField.MASK_SYMBOL) && mistake.getMistakesCount() != 0) {...}
}

class Status {
  //
  if (!gameField.getField().contains(GameField.MASK_SYMBOL) || mistake.getMistakesCount() == 0) {...}
}

class GameField {
  public final static String MASK_SYMBOL = "-";
  //...
  for (int i = 0; i < this.secretWord.length(); i++) {
    this.field += MASK_SYMBOL; 
  }
}
```

**4. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется - в этом случае неважно, будет else или нет, так как программа будет работать одинаково
```
if (!gameField.getField().contains("-") && mistake.getMistakesCount() != 0) {
  return true;
} else {
  return false;
}

//ПРАВИЛЬНО:
if (!gameField.getField().contains("-") && mistake.getMistakesCount() != 0) {
  return true;
} 
return false;
```

**5. Используй примитивные типы**, когда нет необходимости в обертках над примитивами. Тут ее нет
```
public void gerError(Integer errorCount) {
  System.out.println("Такой буквы в слове нет." + " " + "Осталось попыток: " + errorCount);
}

//ПРАВИЛЬНО:
public void gerError(int errorCount)
```

**6. Создавай вспомогательные методы**, делай программу более простой и понятной
```
if (!field.contains(Character.toString(userLetter)) && secretWord.contains(Character.toString(userLetter)))  {...}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняет(/*args or empty*/)) {...}

private boolean isНазваниеКотороеВсеОбъясняет(/*args or empty*/) {
  return !field.contains(Character.toString(userLetter)) && secretWord.contains(Character.toString(userLetter));
}
```

**7. class Dictionary**, словарь слов

- Путь к файлу класс должен получать в конструктор. Иначе класс становится неуниверсальным.
Допустим, в программе будет несколько разных текстовых файлов- на разных языках или по разным темам. 
Тогда вместо нескольких таких специализированных классов Словаря можно будет использовать один универсальный.

- Сейчас класс работает так: при старте создает список слов и устанавливает случайный индекс. Далее при вызове геттера по этому индексу возвращается слово
```
public String getWord() {
  return dictionary.get(randomIndex);
}
```
Зачем сохранять индекс, если можно сохранить слово и выдывать его? Например, так
```
public Dictionary() {
  this.dictionary = new ArrayList<String>();
  this.word = random.nextInt(dictionary.size());
}

public String getWord() {
  return word;
}
``` 

- Но главная особенность класса- после создания он всегда будет выдавать одно и то же слово. Я думаю, через `getWord()` нужно каждый раз выдавать разное случайное слово.

- Игнорировать ошибку чтения файла бессмысленно
```
try {
  this.dictionary = Files.readAllLines(Paths.get("src/main/resources/words.txt"));
} catch (IOException e) {
  e.printStackTrace(); <-- игнорирование ошибки, просто распечатка текста
}
```
Потому что если ошибка действительно произойдет(я удалил words.txt из ресурсов), то программа все равно аварийно вылетит, но по другой причине 
```
Для старта новой игры введите 'Старт'. Для отмены игры введите любой символ
старт
Exception in thread "main" java.lang.IllegalArgumentException: bound must be positive
	at java.base/java.util.Random.nextInt(Random.java:322)
	at src.Dictionary.setWordIndex(Dictionary.java:36)
	at src.Dictionary.<init>(Dictionary.java:19)
	at Hangman.main(Hangman.java:25)
```
Исключение чтения файла здесь нужно не перехватывать. Либо перехватывать и вместо него кидать непроверяемое(unchecked) исключение- так красивее. 
Далее на соответствующем уровне, например в классе Hangman, ловить исключение, писать "Сорян, не удалось прочитать слова" и штатно прекращать работу программы без вываливаня красных ошибок в консоль.

**8. class GameField**

+ 👍 Вариант реализации сущности "маскированное слово".  
Здесь `String secretWord` это текст отгадываемого слова, а `String field`- маска слова.

- В связи с этим метод `resetField()` не сбрасывает маску, а наоборот устанавливает ее: "привет" -> "------"
```
private String field = "";

private void resetField() {
  for (int i = 0; i < this.secretWord.length(); i++) {
    this.field += "-";
  }
}
```

- Нарушение SRP, чужая ответственность, зависимость модели от представления.
Модель(а это модель) не должна ничего печатать в консоль
```
  public void renderField() {
    System.out.println(field);
  }
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах(напр. Андроид) модель нельзя будет использовать. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

- Процесс открывания букв- неочевидный сложный для понимания. Для этого в метод передается не буква, которую нужно открыть, а список индексов букв
```
public void setField(List<Integer> indexes) {
  StringBuilder fieldSB = new StringBuilder(this.field);
  for (Integer index : indexes) {
    fieldSB.setCharAt(index, this.secretWord.charAt(index));
  }
  this.field = fieldSB.toString();
}
```
Тоесть, где-то вовне, вне класса GameField, должен храниться список с индексами открытых букв. 

Для меня логично было бы видеть в классе примерно такой метод
```
public void openLetter(char letter) 
```

- На мой взгляд, в классе не хватает методов:
  - Проверка вхождения буквы в слово
  - Проверка того, что все буквы открыты

**9. class Mistake**

- Две ответственности: счетчик ошибок и хранилище неправильно введенных букв.

- Почему буквы(даже в названии- Letters) хранятся в списке стрингов, а не в списке чаров?
```
private List<String> duplicateWrongLetters;
```

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему.
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6* 
```
private List<String> duplicateWrongLetters;

public List<String> getDuplicateWrongLetters() {
  return duplicateWrongLetters;
}

//ПРАВИЛЬНО:
public List<String> toList() {
  return new ArrayList<>(duplicateWrongLetters);
}
```

**10. class Notifier**

+ 👍 Фактически, отдельный класс вью для распечатки информации. Но раз есть такой красивый класс, выводи через него все сообщения, в том числе распечатывай `GameField`.

**11. class Result**

+ 👍 Отдельный класс для определения выгрыша, это хорошо.

- Из названия метода не видно, на что проверка- на выигрыш, проигрыш, или что-то еще
```
public boolean check(GameField gameField, Mistake mistake) {
  if (!gameField.getField().contains("-") && mistake.getMistakesCount() != 0) {
    return true;
  } else {
    return false;
  }
}

//ПРАВИЛЬНО:
public boolean isWin(GameField gameField, Mistake mistake)
```

- В процессе игры мы все время жолжны проверять, выиграли мы или проиграли. Если есть отдельный класс для проверки результатов игры, в нем кроме метода проверки на выигрыш должен быть метод проверки на проигрыш.

 **12. class Scene**, рендерер хангмана

+ 👍 Отдельный класс для распечатки хангмана это хорошо.

- В любом switch-case должен быть default, в данном случае default должен бросать исключение с сообщением, что такого номера картинки не существует.

- Распечатка картинки висельницы через switch-case или if-elseif - индусский код.
Картинки нужно хранить в статическом массиве и печатать по номеру, например, так
```
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

public void printPicture(int numPicture) {  
  System.out.println(PICTURES[numPicture]);
}
```

**13. class Status**

- Класс среди прочего делит с классом Result ответственность по опредлению результатов игры
```
public void checkGameLoopStatus(GameField gameField, Mistake mistake) {   <-- isGameOver(...)
  if (!gameField.getField().contains("-") || mistake.getMistakesCount() == 0) {
    this.gameLoopStatus = false;
  }
}
```

**14. class Loop**, основная игровая логика

- Класс тяжело читается из-за длинных конструкций. Вводи вспомогательные методы
```
if (!Character.UnicodeBlock.of(userLetter).equals(Character.UnicodeBlock.CYRILLIC))
if (!field.contains(Character.toString(userLetter)) && secretWord.contains(Character.toString(userLetter)))
```

**15. class Hangman**, содержит точку входа в main

- Вместе с классом Loop делит единую ответственность по выполнению основной игровой логики.  
Класс, содержащий точку входа в программу, должен только собирать конфигурацию приложения и запускать ее, но не реализовывать основную логику работы приложения.
В данном случае этот класс должен выглядеть примерно так
```
public class Hangman {
  //...

  public static void main(String[] args) {
    Dictionary dictionary = new Dictionary(FILE_PATCH);

    //где-то тут же в майне должен происходить диалог играть-неиграть

    String word = dictionary.getWord();

    Game game = new Game(word);
    game.start();
  }
}    
```
*Мартин, "ЧК", гл.11, п. "Отделение main"*

## ВЫВОД

Видно, что у автора есть опыт в программировании.

n.76(143)  
#ревью #виселица #оопвиселица