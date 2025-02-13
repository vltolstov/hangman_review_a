https://github.com/GDPixel/HangmanGame 
[Andrey K]

Игра в ООП стиле. Расширеный функционал.  
Уникальная фишка- режим угадывания слова с перемешанными буквами.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. В меню выбора уровня сложности нет пункта "вернуться назад"
```
Change difficulty
────────────────────────────────────────
1 - EASY: 2 open letter(s), 6 health
2 - MEDIUM: 1 open letter(s), 6 health
3 - HARD: 1 open letter(s), 5 health
4 - INSANE: 0 open letter(s), 4 health
────────────────────────────────────────
```

2. Нет вертикальных пробелов между смысловыми блоками
```
You can make 6 more mistake(s)
Word to guess is: _t___
Wrong letters: []
Enter your guess: qqqq
Wrong input. Enter a letter
Word to guess is: _t___
Wrong letters: []
Enter your guess: 

//ЛУЧШЕ:
You can make 6 more mistake(s)
Word to guess is: _t___
Wrong letters: []
Enter your guess: qqqq
Wrong input. Enter a letter

Word to guess is: _t___
Wrong letters: []
Enter your guess: 
```

3. Перечень введенных букв показывается в неправильном порядке
```
Wrong letters: []
Enter your guess: n
<pic>

You can make 5 more mistake(s)
Word to guess is: _o_o_
Wrong letters: [n]
Enter your guess: m
<pic>

You can make 4 more mistake(s)
Word to guess is: _o_o_
Wrong letters: [m, n]
Enter your guess: a
<pic>

You can make 3 more mistake(s)
Word to guess is: _o_o_
Wrong letters: [a, m, n]
Enter your guess: z
<pic>

You can make 2 more mistake(s)
Word to guess is: _o_o_
Wrong letters: [a, z, m, n]
```
Правильно: показывать буквы в порядке ввода или по алфавиту.

## ХОРОШО

1. Красивый пользовательский интерфейс
```
════════════════════════════════════════
            ┓┏
            ┣┫┏┓┏┓┏┓┏┳┓┏┓┏┓
            ┛┗┗┻┛┗┗┫┛┗┗┗┻┛┗
                   ┛
════════════════════════════════════════
              Main menu
────────────────────────────────────────
1 - Start Regular game
2 - Start Scrambled game
3 - Change difficulty
4 - Exit
────────────────────────────────────────
Please enter your choice: 
```
2. Два режима показа слова: обычный и с перемешанными буквами
3. Четыре уровня сложности
4. Можно ввести только одиночную букву английского алфавита
5. Есть список ранее введенных букв

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Название классов слишком похожи. Хотелось бы понимать, чем одна Игра отличается от другой Игры
```
class Game
class HangmanGame
```

- Название вводит в заблуждение. Метод не только показывает меню, но и осуществляет выбор его пунктов
```
void showMainMenu()
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  

**2. class HangedMan**

+ (+)Реализует сущность висельника, которого постепенно вешают и тем самым уменьшают его жизни(health), пока он не умрет. Вынесение этой сущности в отдельный класс это хорошо.

**3. class WrongLetters**

+ (+)Реализует сущность "Хранилище букв", это хорошо.

- Название класса не объясняет суть класса, а объясняет, для чего он используется в программе- для хранения "неправильно введенных букв".
При этом суть класса- просто "хранение букв". Он может хранить правильно введенные, неправильно введенные или все введеные буквы.
Название "неправильно введенные буквы" можно оставить для экземпляра, но не для самого класса. Например `Letters wrongLetters`.

- Нарушение инкапсуляции. Нельзя просто так возвращать Set(List etc.) из класса-обертки
```
public Set<Character> getLetters() {
  return letters;
}
```

Потому что это предоставляет доступ клиенту к внутреннему устрйству обертки
```
public class Main {
  public static void main(String[] args) {
    WrongLetters letters = new WrongLetters();
    letters.addLetter('a');
    letters.addLetter('b');
    System.out.println(letters.getLetters());

    Set set = letters.getLetters();
    set.clear();

    System.out.println(letters.getLetters());
  }
}

//РЕЗУЛЬТАТ
[a, b]
[]
```

В таких случаях нужно возвращать копию хранимого объекта
```
public class WrongLetters {
  //...

  public Set<Character> getLetters() {
    return new HashSet<>(letters);
  }
}

public class Main {...}

//РЕЗУЛЬТАТ
[a, b]
[a, b]
```

**4. class PuzzleWord**

+ (+)Реализует сущность "Секретное слово", это хорошо.

- Название вводит в заблуждение. Говорит, что открывает букву в слове, но открывает букву в маске
```
private final String word;
protected final StringBuilder maskedWord;

protected void openLetterInWord(char letter, String word) {
  //...
  maskedWord.setCharAt(i, letter);
  //...
}

//ПРАВИЛЬНО:
protected void openLetterInMask(char letter, String word) {...}
```

**5. class ScrambledPuzzleWord extends PuzzleWord**

+ (+)Наследник Пазла, где в секретном слове перемешаны буквы. Наследование- хорошо.

**6. class TextFileReader + class WordSelector**

+ (+)Разделение процесса чтения файла и выдачи случайного слова на два класса это хорошо. Теперь можно считывать список слов из разных источников, при этом не меняя алгоритм выдачи случайного слова.
Альтернативный вариант- наследование, например так
```
public abstract class WordRepository {
  public String getRandom() {
    List<String> words = getWords();
    //выдать случайное слово из words
  }
  
  protected abstract List<String> getWords(); 
}

public class FileWordRepository extends WordRepository {
  //...
  @Override
  protected List<String> getWords() {
    //прочитать слова из файла и выдать списком
  }
}

public class InMemoryWordRepository extends WordRepository {
  private final static List<String> WORDS = List.of("инкапсуляция", "наследование", "полиморфизм"); 
  
  @Override
  protected List<String> getWords() {
    return WORDS; 
  }
}
```

**7. Пакет input**

+ (+)Диалоги и Меню- ок.

**8. class HangmanPictures**, распечатка картинок хангмана

- Не вижу причин не делать статическим массив с картинками
```
private final String[] pictures = {...}
```

- Раз в проекте есть отдельный класс HangedMan, который являет собой сущность висельника, логично было бы сделать под него Рендерер
```
public class HangmanPictures {
  private final String[] pictures = {...}

  public void print(int stage) {...}
}

//ПРАВИЛЬНО:
public class HangedManRenderer {
  private static final String[] PICTURES = {...}

  public void print(HangedMan hangedMan) {...}
}
```

**9. class Game**, Игра

+ (+)Все ок.

**10. class HangmanGame**

- Лучше дать другое имя, чтобы не путать с классом `Game`. Например: `GameStarter` или типа того.

- Вот здесь бы пригодились `FileWordRepository` и `InMemoryWordRepository`
```
 try {
   var reader = new TextFileReader(RESOURCE_FILE);
   words = reader.readWords();
 } catch (RuntimeException e) {
   words = new ArrayList<>();
   words.add("elephant");
   words.add("giraffe");
   //oths
 }
```

**11. class Main**, содержит точку входа main

+ (+)Только создает и запускает Запускатор Игры (HangmanGame), это хорошо.

**12. class CustomWordsTestMain**

+ (+)Отдельный майн для тестирвания через заранее известные слова это хорошо
```
public class CustomWordsTestMain {
  public static void main(String[] args) {
    List<String> customWords = new ArrayList<>();
    customWords.add("chair");
    customWords.add("glasses");

    HangmanGame game = new HangmanGame(customWords, GameDifficulty.EASY);
    game.start();
  }
}
```

## ВЫВОД

Получилось реализовать сложный функционал и при этом не сделать архитектуру слишком громоздкой. ООП норм.

n.69(126)  
#ревью #виселица #оопвиселица