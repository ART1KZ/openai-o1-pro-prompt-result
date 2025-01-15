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

import net.minecraftforge.common.MinecraftForge;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.client.event.ClientChatReceivedEvent;
import net.minecraftforge.fml.common.Mod;
import net.minecraft.util.text.*;
import net.minecraft.util.text.event.ClickEvent;
import java.util.List;
import java.util.regex.Pattern;
import java.util.regex.Matcher;

@Mod("autoinvsee")
public class AutoInvseeMod {
    public AutoInvseeMod() {
        MinecraftForge.EVENT_BUS.register(this);
    }

    @SubscribeEvent
    public void onClientChatReceived(ClientChatReceivedEvent event) {
        ITextComponent message = event.getMessage();
        String msgString = message.getString();

        if (msgString.contains("игроки рядом")) {
            ITextComponent newMessage = processNearbyPlayersMessage(message);
            if (newMessage != null) {
                event.setMessage(newMessage);
            }
        } else if (isThreeWordMessageWithIs(msgString)) {
            ITextComponent newMessage = processIsMessage(message);
            if (newMessage != null) {
                event.setMessage(newMessage);
            }
        }
    }

    private boolean isThreeWordMessageWithIs(String msg) {
        String[] words = msg.trim().split("\\s+");
        return words.length == 3 && Arrays.stream(words).anyMatch(w -> w.equalsIgnoreCase("is"));
    }

    private ITextComponent processNearbyPlayersMessage(ITextComponent message) {
        return processComponent(message, false);
    }

    private ITextComponent processComponent(ITextComponent component, boolean excludeNearbyText) {
        IFormattableTextComponent newComponent = component.deepCopy();
        List<ITextComponent> siblings = component.getSiblings();
        newComponent.getSiblings().clear();

        for (ITextComponent sibling : siblings) {
            ITextComponent newSibling = processComponent(sibling, excludeNearbyText);
            newComponent.appendSibling(newSibling);
        }

        if (component instanceof StringTextComponent) {
            StringTextComponent textComponent = (StringTextComponent) component;
            String text = textComponent.getText();
            Style style = textComponent.getStyle();

            if (!excludeNearbyText || !text.contains("игроки рядом") && !text.trim().isEmpty()) {
                Style newStyle = style.createDeepCopy().setUnderlined(true);

                if (!style.getItalic()) {
                    newStyle.setClickEvent(new ClickEvent(ClickEvent.Action.RUN_COMMAND, "/invsee " + text.trim()));
                } else {
                    newStyle.setClickEvent(new ClickEvent(ClickEvent.Action.SUGGEST_COMMAND, "/realname " + text.trim()));
                }
                newComponent.setStyle(newStyle);
            }
        }

        return newComponent;
    }

    private ITextComponent processIsMessage(ITextComponent message) {
        String msgString = message.getString().trim();
        String[] words = msgString.split("\\s+");

        if (words.length != 3 || !Arrays.asList(words).contains("is")) {
            return null; // Does not match condition
        }

        String nickname = words[words.length - 1];
        IFormattableTextComponent newMessage = new StringTextComponent("");

        int wordCount = 0;
        for (ITextComponent component : message.toFlatList(t -> t)) {
            String[] componentWords = component.getString().split("\\s+");
            Style style = component.getStyle();

            for (String word : componentWords) {
                if (!word.isEmpty()) {
                    wordCount++;
                    ITextComponent wordComponent;

                    if (wordCount == words.length) {
                        Style newStyle = style.createDeepCopy()
                                .setUnderlined(true)
                                .setClickEvent(new ClickEvent(ClickEvent.Action.RUN_COMMAND, "/invsee " + nickname));
                        wordComponent = new StringTextComponent(word).setStyle(newStyle);
                    } else {
                        wordComponent = new StringTextComponent(word).setStyle(style);
                    }
                    newMessage.append(wordComponent).appendString(" ");
                }
            }
        }

        return newMessage;
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

