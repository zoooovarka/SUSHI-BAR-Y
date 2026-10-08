# Суши-шеф

Кулинарный тайм-менеджмент для Яндекс Игр. Игрок — повар: посетители входят в зал, встают к стойке и заказывают еду, игрок готовит и подаёт блюда, пока у них не кончилось терпение. Между уровнями кухня улучшается, а за звёзды открываются новые рестораны азиатской кухни.

Весь код — один файл `index.html` (HTML + CSS + JS, без сборщиков и библиотек). Картинки кладутся в `img/`; если файла нет, рисуется эмодзи-заглушка, так что игра работает и без графики.

## Запуск

```bash
python3 -m http.server 8000
# открыть http://localhost:8000/
```

Локально `/sdk.js` нет, поэтому игра работает на заглушках SDK: сохранения в `localStorage`, ролик за вознаграждение имитируется секундной заставкой, вместо полноэкранной рекламы тоже секундная заставка, покупка подтверждается окном браузера.

Для Яндекс Игр: заархивируйте `index.html` и папку `img/` (в корне архива). В консоли разработчика создайте лидерборд с техническим именем `hearts` (задаётся в `CONFIG.leaderboard`).

## Как устроено

- **DOM, а не canvas**: браузер сам определяет, по какому элементу пришёлся тап. Сцена 1600×900 (16:9) масштабируется под экран; на телефоне в портретной ориентации игра встаёт на паузу и просит повернуть устройство.
- **Ввод**: кухня реагирует на `pointerdown` (без задержки, работает мультитач), кнопки интерфейса — на `click`. Перетаскивания нет.
- **CONFIG** описывает спрайты, рестораны, блюда, станции, рецепты, улучшения, уровни и 15 достижений. Логика знает только четыре типа станций: `assembly` (циновки), `fryer` (фритюр), `dispenser` (кастрюля, чайник), `trash` (мусорка). Новый ресторан — новая запись в `CONFIG.restaurants`.
- **Улучшения** задают параметры по ступеням (`params`), а станции ссылаются на эти параметры по имени. Поле станции `look` указывает, от какого улучшения зависит её картинка: `mat_0…mat_3`, `fryer_0…3`, `pot_0…3`, `teapot_0…3`. Без картинок оборудование меняет рамку: дерево → бронза → серебро → золото.
- **Тексты** на русском и английском в объекте `TEXT`, язык берётся из `ysdk.environment.i18n.lang`.

## Механики

- Посетитель входит слева, идёт к свободному месту, ждёт с пузырём заказа и полоской терпения. Обслуженный уходит направо довольным, необслуженный — налево сердитым.
- **Роллы**: тап по нори кладёт основу на свободную циновку, тап по начинке кладёт её на первую основу без неё, тап по циновке скручивает ролл (2 с).
- **Мисо-суп**: тап по миске ставит её на подставку, тофу и вакамэ добавляются так же, как начинки роллов, тап по миске наливает бульон (2 с). Если в миске только одна добавка, получится «неправильный» суп — его можно только выбросить.
- **Темпура**: креветка жарится 5 с, через 6 с после готовности сгорает (штраф 5 при выбрасывании). Тап по готовой снимает её с огня на поднос.
- **Чай**: тап по чайнику ставит его кипятиться (4 с), после этого каждый тап по чашке наливает порцию в лоток. Вскипевшей воды хватает на 2 чашки (с улучшениями — до 4), потом чайник надо кипятить снова.
- **Подача**: тап по готовому блюду выбирает его (жёлтая рамка). Посетители, которым оно нужно, подсвечиваются зелёным, а кому отдать — решает игрок тапом по посетителю. Отдать не тому нельзя: пузырь трясётся.
- **Мусорка**: если блюдо выбрано, тап по мусорке выбрасывает его. Без выбора тап включает режим выбрасывания: следующий тап по слоту очищает его. Сгоревшее и неправильное выбрасывается и обычным тапом.
- **Уровни**: в суши-баре 10 уровней. На уровнях 1–7 по одному добавляются блюда (лосось, мисо, огурец, темпура, авокадо, ассорти, чай), на 8–10 растут темп и длина заказов. Уровни 1–3 обучающие. Список уровней — `levels` в CONFIG, их число можно менять: карта уровней раскладывается рядами по 5.
- **Гарантия гостей** (`CONFIG.guestPlan`): гости приходят случайно (интервал `every` уровня), но до последних 20% времени уровня (не меньше 12 с) они обязательно закажут блюд минимум на 90% от цели 3★ по цене меню. Остальное добирается чаевыми и комбо. Если случайный поток отстаёт, гости приходят чаще (не чаще раза в 1,5 с) и заказывают по максимуму позиций. Гарантия не обходит занятые места: если все места заняты, новый гость ждёт.
- **«Официант»** (зал, одна ступень, 5000 монет в каждом ресторане): тап по готовому блюду сразу отдаёт его посетителю, который ждёт дольше всех, а готовое с огня уходит прямо гостю. Если блюдо никому не нужно, оно выделяется, и тап по мусорке его выбрасывает. За рекламу его получить нельзя (`noAd`). Вместо монет можно купить за реальные деньги (`product: 'waiter'`): покупка открывает «Официанта» сразу во всех ресторанах навсегда.
- **Рестораны**: пять ресторанов по 10 уровней (30 звёзд), требования — 0, 20, 45, 70 и 95 звёзд. В каждом 7 блюд и те же четыре типа станций:
  - «Суши-бар»: роллы, темпура, мисо-суп, зелёный чай;
  - «Раменная»: рамен 4 видов, гёдза на сковороде, эдамамэ в кастрюле, ячменный чай из кувшина;
  - «Вок-стрит»: лапша вок 4 видов, спринг-роллы во фритюре, рис из рисоварки в коробочке, бабл-ти (стакан чая + тапиока);
  - «Чайная сладостей»: дайфуку 4 видов, данго на гриле, тайяки в форме-рыбке, матча-латте;
  - «Димсам-хаус»: корзинки с хар гау, шумай, сяолунбао и ассорти, баоцзы в большой пароварке, яичные тарталетки в духовке, жасминовый чай.
- **Реклама**: после каждого пройденного уровня — полноэкранная реклама, затем окно итогов (монеты и звёзды зачисляются до рекламы). После проигрыша реклама показывается при выходе из окна итогов. Яндекс сам не показывает полноэкранную рекламу чаще раза в минуту; в этом случае игра просто идёт дальше. Без SDK (локально, на GitHub Pages) вместо рекламы на секунду появляется заставка «тестовая заглушка».
- **Покупка «Официанта»**: в консоли Яндекс Игр (раздел «Покупки») нужно включить покупки и создать товар с ID `waiter` и ценой 250 ₽. Покупка нерасходуемая: при запуске игра запрашивает `getPurchases()` и восстанавливает её, поэтому «Официант» не пропадает при смене устройства. Цена на кнопке берётся из каталога Яндекса (`getCatalog()`), а `CONFIG.products.waiter.price` — запасная надпись. Без SDK покупка подтверждается обычным окном браузера, в режиме `?test` проходит сразу.
- **Звук**: музыка и эффекты из папки `sounds/` (список файлов и советы — в [`sounds/README.md`](sounds/README.md)). Без файлов играет синтезированный звук, как раньше. На карте играет трек `music_map.mp3`, на уровне — `music_game.mp3`; после победы, вместе с окном звёзд, звучит `yooo.mp3`.
- Общие параметры зала (посетители, чаевые, комбо) вынесены в `HALL`, каждый ресторан подключает их через `...HALL`.
- **Ширина кухни**: если станций много (поздние уровни с полной прокачкой), кухня целиком ужимается, чтобы поместиться по ширине.
- Оплата за весь заказ; чаевые +50% при терпении выше 66% и +20% выше 33%. Сердце, если терпение выше 50%.
- Комбо: каждая подача не позже чем через 3 с после предыдущей поднимает множитель x2 → x3 → x4 и даёт +3/+6/+10 монет. Комбо показывается крупной надписью и счётчиком в шапке.
- Достижения открываются автоматически, награду игрок забирает в окне достижений (кнопка с медалью на карте).

## Баланс

Пороги звёзд подобраны ботом: он играет по той же логике, что рука-подсказка, по 3 раза на каждый уровень в 5 профилях (реакция 0,6 с или 1,0 с; без улучшений, с первой ступенью всех улучшений, с полной прокачкой). Для 1★ хватает медленной игры без улучшений, для 2★ нужна быстрая игра, для 3★ на поздних уровнях нужны и скорость, и улучшения.

**Тестовый режим:** `index.html?test` — 99999 монет и все уровни открыты, на карте и карте уровней видна метка «ТЕСТ». Прогресс хранится в отдельном сохранении (`sushi_chef_save_v1_test`), в облако Яндекса и в лидерборд ничего не уходит, обычное сохранение не меняется. Монеты пополняются до 99999 при каждой загрузке. Режим сочетается с отладкой: `?test&debug&bot&speed=5`.

Режим отладки: `index.html?debug&bot&speed=10&botdt=0.6` — бот играет сам, `speed` ускоряет время, `botdt` задаёт паузу между его действиями. Без `?debug` всё это выключено.

## Спрайты

План ресторанов «Раменная», «Вок-стрит», «Чайная сладостей» и «Димсам-хаус» и нужные для них картинки — в [`docs/restaurants-sprites.md`](docs/restaurants-sprites.md).

Все файлы необязательны: если картинки нет, игра рисует эмодзи-заглушку. Фоны в `.jpg`, остальное в `.png` с прозрачностью (палитра 256 цветов, чтобы архив был лёгким).

Исходные листы художника лежат в `art/`. В архив для Яндекс Игр они не нужны: игра берёт только `index.html` и `img/`. Из листов нарезаны спрайты: свечение вокруг предметов убрано, соседние фигуры разделены по контуру.

- Картинка суши-бара разрезана на три слоя: `bg_hall_sushi.jpg` (зал позади посетителей), `counter_sushi.jpg` (стойка поверх посетителей) и `bg_kitchen_sushi.jpg` (столешница под кухней). Из неё же сделаны размытые фоны карты уровней и магазина.
- Карта островов — `bg_world.jpg`. Если она загрузилась, рестораны показываются табличками над нарисованными домиками (координаты в `mapPos`), иначе — кругами с эмодзи.
- Оборудование выбирается по ступени улучшения: `mat_0…3` → `mat.png`, `fryer_0…1` → `fryer.png`, `fryer_2…3` → `fryer_gold.png`. Рамка вокруг циновок, фритюра, кастрюли и чайника тоже меняется: дерево → бронза → серебро → золото.
- `roll_wrong.png` (неправильный ролл) собран из ролла-ассорти: обесцвечен и перечёркнут.
- Рука-указатель на картинке смотрит вниз; где у неё кончик пальца, задаёт `CONFIG.handTip`.

### Уже в `img/` (219)

| Файл | Ключи в CONFIG | Заглушка |
|---|---|---|
| `logo.png` | logo | 🍣 |
| `coin.png` | coin | 🪙 |
| `heart.png` | heart | ❤️ |
| `star.png` | star | ⭐ |
| `star_empty.png` | star_empty | ⭐ |
| `lock.png` | lock | 🔒 |
| `hand.png` | hand | 👆 |
| `combo.png` | combo | 🔥 |
| `medal.png` | medal | 🏅 |
| `ic_pause.png` | ic_pause | ⏸️ |
| `ic_sound.png` | ic_sound | 🔊 |
| `ic_mute.png` | ic_mute | 🔇 |
| `ic_shop.png` | ic_shop | 🛒 |
| `ic_gift.png` | ic_gift | 🎁 |
| `ic_trophy.png` | ic_trophy | 🏆 |
| `ic_back.png` | ic_back | ⬅️ |
| `ic_ad.png` | ic_ad | 📺 |
| `ic_clock.png` | ic_clock | ⏱️ |
| `ic_close.png` | ic_close | ✖️ |
| `bg_world.jpg` | bg_world | CSS |
| `bg_levels_sushi.jpg` | bg_levels_sushi | CSS |
| `bg_hall_sushi.jpg` | bg_hall_sushi | CSS |
| `bg_kitchen_sushi.jpg` | bg_kitchen_sushi | CSS |
| `bg_shop.jpg` | bg_shop | CSS |
| `counter_sushi.jpg` | counter_sushi | CSS |
| `visitor_1.png` | visitor_1 | 👦 |
| `visitor_1_angry.png` | visitor_1_angry | 😠 |
| `visitor_2.png` | visitor_2 | 👩 |
| `visitor_2_angry.png` | visitor_2_angry | 😠 |
| `visitor_3.png` | visitor_3 | 👴 |
| `visitor_3_angry.png` | visitor_3_angry | 😠 |
| `visitor_4.png` | visitor_4 | 👧 |
| `visitor_4_angry.png` | visitor_4_angry | 😠 |
| `visitor_5.png` | visitor_5 | 🧔 |
| `visitor_5_angry.png` | visitor_5_angry | 😠 |
| `visitor_6.png` | visitor_6 | 👵 |
| `visitor_6_angry.png` | visitor_6_angry | 😠 |
| `mood_happy.png` | mood_happy | 😊 |
| `mood_angry.png` | mood_angry | 😠 |
| `nori_rice.png` | nori | 🍙 |
| `salmon.png` | salmon | 🐟 |
| `cucumber.png` | cucumber | 🥒 |
| `avocado.png` | avocado | 🥑 |
| `roll_salmon.png` | roll_salmon | 🍣 + 🐟 |
| `roll_cucumber.png` | roll_cucumber | 🍣 + 🥒 |
| `roll_avocado.png` | roll_avocado | 🍣 + 🥑 |
| `roll_mix.png` | roll_mix | 🍣 + 🌈 |
| `roll_wrong.png` | roll_wrong | 🍣 + ❌ |
| `shrimp_raw.png` | shrimp_raw | 🦐 |
| `tempura.png` | tempura | 🍤 |
| `tempura_burnt.png` | tempura_burnt | 🍤 |
| `miso.png` | miso | 🍲 |
| `miso_wrong.png` | miso_wrong | 🍲 + ❌ |
| `bowl.png` | bowl | 🥣 |
| `tofu.png` | tofu | 🧊 |
| `wakame.png` | wakame | 🌿 |
| `cup_empty.png` | cup_empty | 🍵 |
| `tea.png` | tea | 🍵 |
| `mat.png` | mat_0, mat_1, mat_2, mat_3 | CSS |
| `fryer.png` | fryer_0, fryer_1 | CSS |
| `fryer_gold.png` | fryer_2, fryer_3 | CSS |
| `pot.png` | pot_0, pot_1, pot_2, pot_3 | 🥘 |
| `teapot.png` | teapot_0, teapot_1, teapot_2, teapot_3 | 🫖 |
| `trash.png` | trash | 🗑️ |
| `rest_sushi.png` | rest_sushi | 🍣 |
| `rest_ramen.png` | rest_ramen | 🍜 |
| `rest_wok.png` | rest_wok | 🥡 |
| `rest_dimsum.png` | rest_dimsum | 🥟 |
| `rest_sweets.png` | rest_sweets | 🍡 |
| `up_roll_speed.png` | up_roll_speed | ⚡ |
| `up_mats.png` | up_mats | 🎋 |
| `up_fryer_speed.png` | up_fryer_speed | 🔥 |
| `up_fryer_slots.png` | up_fryer_slots | 🍳 |
| `up_soup_cap.png` | up_soup_cap | 🥣 |
| `up_tea_cap.png` | up_tea_cap | 🫖 |
| `up_price_roll.png` | up_price_roll | 🍣 |
| `up_price_tempura.png` | up_price_tempura | 🍤 |
| `up_price_soup.png` | up_price_soup | 🍲 |
| `up_patience.png` | up_patience | ⌛ |
| `up_tips.png` | up_tips | 💰 |
| `up_auto_serve.png` | up_auto_serve | 🛎️ |
| `bg_levels_ramen.jpg` | bg_levels_ramen | CSS |
| `bg_hall_ramen.jpg` | bg_hall_ramen | CSS |
| `bg_kitchen_ramen.jpg` | bg_kitchen_ramen | CSS |
| `counter_ramen.jpg` | counter_ramen | CSS |
| `ramen_base.png` | ramen_base | 🍜 |
| `chashu.png` | chashu | 🥓 |
| `ajitama.png` | ajitama | 🥚 |
| `corn.png` | corn | 🌽 |
| `ramen_chashu.png` | ramen_chashu | 🍜 + 🥓 |
| `ramen_egg.png` | ramen_egg | 🍜 + 🥚 |
| `ramen_corn.png` | ramen_corn | 🍜 + 🌽 |
| `ramen_full.png` | ramen_full | 🍜 + 🌈 |
| `ramen_wrong.png` | ramen_wrong | 🍜 + ❌ |
| `gyoza_raw.png` | gyoza_raw | 🥟 |
| `gyoza.png` | gyoza | 🥟 + 🔥 |
| `gyoza_burnt.png` | gyoza_burnt | 🥟 |
| `pan.png` | pan_0, pan_1 | CSS |
| `pan_gold.png` | pan_2, pan_3 | CSS |
| `edamame_raw.png` | edamame_raw | 🫛 |
| `edamame.png` | edamame | 🫛 + 🥣 |
| `edamame_over.png` | edamame_over | 🫛 |
| `boil_pot.png` | boilpot_0, boilpot_1 | CSS |
| `boil_pot_gold.png` | boilpot_2, boilpot_3 | CSS |
| `jug.png` | jug_0, jug_1, jug_2, jug_3 | 🫖 |
| `glass_empty.png` | glass_empty | 🥛 |
| `mugicha.png` | mugicha | 🥤 |
| `up_broth_speed.png` | up_broth_speed | ⚡ |
| `up_ramen_slots.png` | up_ramen_slots | 🍜 |
| `up_pan_speed.png` | up_pan_speed | 🔥 |
| `up_pan_slots.png` | up_pan_slots | 🍳 |
| `up_boil_slots.png` | up_boil_slots | ♨️ |
| `up_jug.png` | up_jug | 🫖 |
| `up_price_ramen.png` | up_price_ramen | 🍜 |
| `up_price_snack.png` | up_price_snack | 🥟 |
| `up_price_drink.png` | up_price_drink | 🥤 |
| `bg_levels_wok.jpg` | bg_levels_wok | CSS |
| `bg_hall_wok.jpg` | bg_hall_wok | CSS |
| `bg_kitchen_wok.jpg` | bg_kitchen_wok | CSS |
| `counter_wok.jpg` | counter_wok | CSS |
| `wok_noodles.png` | wok_noodles | 🍝 |
| `chicken.png` | chicken | 🍗 |
| `wok_shrimp.png` | wok_shrimp | 🦐 |
| `veggies.png` | veggies | 🥦 |
| `wok_chicken.png` | wok_chicken | 🥡 + 🍗 |
| `wok_shrimp_dish.png` | wok_prawn | 🥡 + 🦐 |
| `wok_veg.png` | wok_veg | 🥡 + 🥦 |
| `wok_mix.png` | wok_mix | 🥡 + 🌈 |
| `wok_wrong.png` | wok_wrong | 🥡 + ❌ |
| `wok_pan.png` | wokpan_0, wokpan_1, wokpan_2, wokpan_3 | CSS |
| `springroll_raw.png` | springroll_raw | 🌯 |
| `springroll.png` | springroll | 🌯 + 🔥 |
| `springroll_burnt.png` | springroll_burnt | 🌯 |
| `rice_cooker.png` | ricecooker_0, ricecooker_1, ricecooker_2, ricecooker_3 | 🍚 |
| `box_empty.png` | box_empty | 🥡 |
| `rice_box.png` | rice_box | 🍚 |
| `bubble_cup.png` | bubble_cup | 🥛 |
| `tapioca.png` | tapioca | ⚫ |
| `bubble_tea.png` | bubble_tea | 🧋 |
| `bubble_wrong.png` | bubble_wrong | 🧋 + ❌ |
| `up_wok_speed.png` | up_wok_speed | ⚡ |
| `up_wok_slots.png` | up_wok_slots | 🥘 |
| `up_wok_fryer_speed.png` | up_wok_fryer_speed | 🔥 |
| `up_wok_fryer_slots.png` | up_wok_fryer_slots | 🍳 |
| `up_rice_cooker.png` | up_rice_cooker | 🍚 |
| `up_bubble_slots.png` | up_bubble_slots | 🧋 |
| `up_price_wok.png` | up_price_wok | 🥡 |
| `up_price_springroll.png` | up_price_springroll | 🌯 |
| `up_price_bubble.png` | up_price_bubble | 🧋 |
| `bg_levels_sweets.jpg` | bg_levels_sweets | CSS |
| `bg_hall_sweets.jpg` | bg_hall_sweets | CSS |
| `bg_kitchen_sweets.jpg` | bg_kitchen_sweets | CSS |
| `counter_sweets.jpg` | counter_sweets | CSS |
| `mochi_dough.png` | mochi_dough | ⚪ |
| `anko.png` | anko | 🫘 |
| `strawberry.png` | strawberry | 🍓 |
| `matcha_cream.png` | matcha_cream | 🍵 |
| `daifuku_anko.png` | daifuku_anko | 🍡 + 🫘 |
| `daifuku_strawberry.png` | daifuku_strawberry | 🍡 + 🍓 |
| `daifuku_matcha.png` | daifuku_matcha | 🍡 + 🍵 |
| `daifuku_mix.png` | daifuku_mix | 🍡 + 🌈 |
| `daifuku_wrong.png` | daifuku_wrong | 🍡 + ❌ |
| `dango_raw.png` | dango_raw | 🍡 |
| `dango.png` | dango | 🍡 + 🔥 |
| `dango_burnt.png` | dango_burnt | 🍡 |
| `grill.png` | grill_0, grill_1 | CSS |
| `grill_gold.png` | grill_2, grill_3 | CSS |
| `taiyaki_batter.png` | taiyaki_batter | 🥛 |
| `taiyaki.png` | taiyaki | 🐟 |
| `taiyaki_burnt.png` | taiyaki_burnt | 🐟 |
| `taiyaki_mold.png` | tmold_0, tmold_1 | CSS |
| `taiyaki_mold_gold.png` | tmold_2, tmold_3 | CSS |
| `matcha_kettle.png` | mkettle_0, mkettle_1, mkettle_2, mkettle_3 | 🫖 |
| `latte_cup_empty.png` | latte_cup_empty | 🍵 |
| `matcha_latte.png` | matcha_latte | 🍵 |
| `up_daifuku_speed.png` | up_daifuku_speed | ⚡ |
| `up_daifuku_slots.png` | up_daifuku_slots | 🍡 |
| `up_grill_speed.png` | up_grill_speed | 🔥 |
| `up_grill_slots.png` | up_grill_slots | 🍢 |
| `up_taiyaki_slots.png` | up_taiyaki_slots | 🐟 |
| `up_matcha_kettle.png` | up_matcha_kettle | 🫖 |
| `up_price_daifuku.png` | up_price_daifuku | 🍡 |
| `up_price_sweets.png` | up_price_sweets | 🍢 |
| `up_price_latte.png` | up_price_latte | 🍵 |
| `bg_levels_dimsum.jpg` | bg_levels_dimsum | CSS |
| `bg_hall_dimsum.jpg` | bg_hall_dimsum | CSS |
| `bg_kitchen_dimsum.jpg` | bg_kitchen_dimsum | CSS |
| `counter_dimsum.jpg` | counter_dimsum | CSS |
| `steamer_empty.png` | steamer_empty | 🧺 |
| `hargau.png` | hargau | 🥟 |
| `siumai.png` | siumai | 🥟 |
| `xlb.png` | xlb | 🥟 |
| `steamer_hargau.png` | steamer_hargau | 🧺 + 🦐 |
| `steamer_siumai.png` | steamer_siumai | 🧺 + 🟡 |
| `steamer_xlb.png` | steamer_xlb | 🧺 + 🥟 |
| `steamer_mix.png` | steamer_mix | 🧺 + 🌈 |
| `steamer_wrong.png` | steamer_wrong | 🧺 + ❌ |
| `bao_raw.png` | bao_raw | ⚪ |
| `bao.png` | bao | 🥟 + ♨️ |
| `bao_over.png` | bao_over | 🥟 |
| `steamer_big.png` | bigsteamer_0, bigsteamer_1 | CSS |
| `steamer_big_gold.png` | bigsteamer_2, bigsteamer_3 | CSS |
| `tart_raw.png` | tart_raw | 🥧 |
| `egg_tart.png` | egg_tart | 🥧 + 🔥 |
| `tart_burnt.png` | tart_burnt | 🥧 |
| `oven.png` | oven_0, oven_1 | CSS |
| `oven_gold.png` | oven_2, oven_3 | CSS |
| `clay_teapot.png` | claypot_0, claypot_1, claypot_2, claypot_3 | 🫖 |
| `tea_cup_small.png` | tea_cup_small | 🍵 |
| `jasmine_tea.png` | jasmine_tea | 🍵 |
| `up_steam_speed.png` | up_steam_speed | ⚡ |
| `up_steamer_slots.png` | up_steamer_slots | 🧺 |
| `up_bao_slots.png` | up_bao_slots | ♨️ |
| `up_oven_speed.png` | up_oven_speed | 🔥 |
| `up_oven_slots.png` | up_oven_slots | 🥧 |
| `up_clay_teapot.png` | up_clay_teapot | 🫖 |
| `up_price_dimsum.png` | up_price_dimsum | 🥟 |
| `up_price_bakery.png` | up_price_bakery | 🥧 |
| `up_price_tea.png` | up_price_tea | 🍵 |

Все картинки из `CONFIG.sprites` на месте; если какой-то файл удалить, игра нарисует вместо него эмодзи-заглушку.
