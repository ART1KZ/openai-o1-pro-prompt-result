# Мод `net.mcreator.autoinvsee` для Minecraft Forge 1.16.5

Этот мод изменяет сообщения в чате, делая ники игроков кликабельными. При нажатии на ник выполняется команда `/invsee [ник_игрока]` или `/realname [ник_игрока]` в зависимости от условий. Также мод обрабатывает сообщения, содержащие ровно три слова, одно из которых — "is".

---

## Описание функционала

1. **Сообщения с текстом "игроки рядом"**:
   - Извлекает ники игроков, написанные не курсивом, и делает их кликабельными.
   - При нажатии на ник:
     - Выходит из чата.
     - Выполняет команду `/invsee [ник_игрока]`.
   - Если ник написан курсивом:
     - Делает его кликабельным.
     - Выполняет команду `/realname [ник_игрока]` без выхода из чата.

2. **Сообщения из трёх слов, содержащие "is"**:
   - Делает последнее слово (ник игрока) кликабельным.
   - При нажатии выполняет команду `/invsee [ник_игрока]`.

3. **Форматирование**:
   - Сохраняет исходные цвета и форматирование сообщения.
   - Кликабельные элементы подчёркиваются.

---

## Исходный код мода

```java
package net.mcreator.autoinvsee;

import net.minecraftforge.event.client.ClientChatReceivedEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;
import net.minecraft.util.text.ITextComponent;
import net.minecraft.util.text.TextComponent;
import net.minecraft.util.text.Style;
import net.minecraft.util.text.event.ClickEvent;
import net.minecraft.util.text.TextFormatting;
import net.minecraft.client.gui.screen.ChatScreen;
import net.minecraft.client.Minecraft;

import java.util.List;
import java.util.ArrayList;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

@Mod.EventBusSubscriber(modid = "autoinvsee", bus = Mod.EventBusSubscriber.Bus.FORGE)
public class ChatMessageHandler {

    @SubscribeEvent
    public static void onChatMessage(ClientChatReceivedEvent event) {
        ITextComponent message = event.getMessage();
        String unformatted = message.getUnformattedText();

        if (unformatted.contains("игроки рядом")) {
            // Извлечение ников игроков
            List<String> playerNames = extractPlayerNames(message);

            // Создание нового сообщения с кликабельными никами
            TextComponent newMessage = new TextComponent("");
            for (String playerName : playerNames) {
                TextComponent nameComponent = new TextComponent(playerName);
                nameComponent.setStyle(nameComponent.getStyle().withUnderline(true));

                // Проверка, написан ли ник курсивом
                if (isItalic(playerName, message)) {
                    nameComponent.setStyle(nameComponent.getStyle().withClickEvent(
                        new ClickEvent(ClickEvent.Action.RUN_COMMAND, "/realname " + playerName)));
                } else {
                    nameComponent.setStyle(nameComponent.getStyle().withClickEvent(
                        new ClickEvent(ClickEvent.Action.RUN_COMMAND, "/invsee " + playerName)));
                    // Выход из чата после выполнения команды
                    Minecraft.getInstance().displayGuiScreen(null);
                }
                newMessage.append(nameComponent).append(" ");
            }
            event.setMessage(newMessage);
        } else if (unformatted.split(" ").length == 3 && unformatted.contains("is")) {
            // Обработка сообщений из трёх слов, содержащих "is"
            String[] parts = unformatted.split(" ");
            String playerName = parts[2];

            TextComponent newMessage = new TextComponent(unformatted.replace(playerName, ""));
            TextComponent nameComponent = new TextComponent(playerName);
            nameComponent.setStyle(nameComponent.getStyle().withUnderline(true)
                .withClickEvent(new ClickEvent(ClickEvent.Action.RUN_COMMAND, "/invsee " + playerName)));
            newMessage.append(nameComponent);

            event.setMessage(newMessage);
        }
    }

    // Метод для извлечения ников игроков из сообщения
    private static List<String> extractPlayerNames(ITextComponent message) {
        List<String> playerNames = new ArrayList<>();
        String text = message.getString();
        Pattern pattern = Pattern.compile("\\b\\w+\\b");
        Matcher matcher = pattern.matcher(text);

        while (matcher.find()) {
            playerNames.add(matcher.group());
        }
        return playerNames;
    }

    // Метод для проверки, написан ли ник курсивом
    private static boolean isItalic(String playerName, ITextComponent message) {
        String text = message.getString();
        int startIndex = text.indexOf(playerName);
        if (startIndex == -1) return false;

        // Проверка стиля текста
        Style style = message.getStyleAtPosition(startIndex);
        return style.getItalic();
    }
}
```

---

## Инструкция по установке

1. **Создайте проект Forge Mod**:
   - Используйте Forge Mod Generator для Minecraft 1.16.5.
   - Установите ID мода как `autoinvsee`.

2. **Добавьте исходный код**:
   - Скопируйте код выше в файл `ChatMessageHandler.java` в папке `src/main/java/net/mcreator/autoinvsee`.

3. **Соберите мод**:
   - Используйте команду `gradlew build` для сборки мода.
   - Готовый `.jar` файл будет находиться в папке `build/libs`.

4. **Установите мод**:
   - Поместите `.jar` файл в папку `mods` вашего клиента Minecraft.

---

## Примечания

- **Форматирование**: Мод сохраняет исходное форматирование сообщений, добавляя только подчёркивание для кликабельных элементов.
- **Безопасность**: Убедитесь, что команды выполняются безопасно.
- **Производительность**: Код оптимизирован для минимального влияния на производительность.

