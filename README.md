3. ПРОГРАММНАЯ РЕАЛИЗАЦИЯ
3.1 Выбор среды разработки и языка программирования
3.1.1 Обоснование выбора языка программирования

Для реализации алгоритма шифрования «Кузнечик» был выбран язык программирования Python 3.10+ по следующим причинам:
Критерий	Обоснование
Кроссплатформенность	Python работает на всех основных операционных системах (Windows, Linux, macOS) без изменений кода
Богатая стандартная библиотека	Наличие модулей для работы с байтами (bytes, bytearray), хеширования (hashlib), файлового ввода/вывода, что упрощает реализацию
Простота отладки	Интерактивный режим, подробные сообщения об ошибках, возможность пошагового выполнения
Читаемость кода	Высокоуровневый синтаксис позволяет сосредоточиться на алгоритме, а не на деталях реализации
Быстрая разработка	Динамическая типизация и отсутствие необходимости компиляции ускоряют процесс разработки
Объектно-ориентированный подход	Возможность создания четкой структуры классов с инкапсуляцией функциональности
Поддержка работы с битами	Наличие побитовых операций (&, |, ^, <<, >>), необходимых для криптографических преобразований
3.1.2 Среда разработки

Для разработки использовалась следующая конфигурация:
Компонент	Версия	Назначение
Python	3.10+	Интерпретатор языка
Visual Studio Code	1.86+	Редактор кода с поддержкой Python
Расширения VS Code	Python, Pylance	Автодополнение, проверка синтаксиса
Git	2.40+	Система контроля версий
ОС разработки	Windows 11 / Linux (тестирование)	Кроссплатформенная разработка
3.1.3 Структура проекта
text

kuznechik-cipher/
│
├── kuznechik.py              # Основной файл программы (полная реализация)
├── README.md                  # Документация по использованию
├── requirements.txt           # Зависимости (при их наличии)
│
├── keys/                       # Директория для хранения ключей
│   ├── key_20260315_161635.key # Пример сохраненного ключа
│   └── key_20260315_172528.key # Пример сохраненного ключа
│
├── examples/                   # Примеры для тестирования
│   ├── OpenText.txt            # Исходный текст
│   ├── CryptText.txt           # Зашифрованный текст
│   └── decrypt.txt             # Расшифрованный текст
│
└── docs/                        # Документация
    └── report.md                # Отчет о разработке

3.2 Архитектура программной реализации
3.2.1 Общая архитектура

Программа реализована в объектно-ориентированном стиле и состоит из трех основных классов, взаимодействующих между собой:
text

┌─────────────────────────────────────────────────────────────────┐
│                      KUZNECHIK (Ядро алгоритма)                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ • S-блоки (прямой и обратный)                             │  │
│  │ • Линейное преобразование L (16x R)                       │  │
│  │ • Умножение в поле Галуа GF(2^8)                           │  │
│  │ • Генерация раундовых ключей (сеть Фейстеля)               │  │
│  │ • Шифрование/расшифрование блоков                          │  │
│  │ • Режимы: ECB, CBC, CFB, OFB, CTR                          │  │
│  │ • Вычисление и проверка имитовставки (MAC)                 │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      KEYMANAGER                                  │
│              (Управление ключами шифрования)                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ • Сохранение ключей в файлы (директория keys/)             │  │
│  │ • Загрузка ключей из файлов                                │  │
│  │ • Просмотр списка сохраненных ключей                       │  │
│  │ • Поддержка ключей разного формата (32/48 байт)            │  │
│  │ • Генерация ключей из пароля (50 итераций SHA-512)         │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    INTERACTIVEMENU                               │
│              (Интерактивный пользовательский интерфейс)         │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ • Главное меню с навигацией по номерам                     │  │
│  │ • Меню выбора режима шифрования                            │  │
│  │ • Меню настройки имитовставки (MAC)                        │  │
│  │ • Меню установки синхропосылки (IV)                        │  │
│  │ • Меню управления ключами                                  │  │
│  │ • Меню шифрования файлов                                   │  │
│  │ • Меню расшифрования файлов                                │  │
│  │ • Тестовый цикл для проверки всех режимов                  │  │
│  │ • Статистика работы                                        │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘

3.2.2 Диаграмма потоков данных
text

┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Входной   │────▶│  Дополнение │────▶│  Разбиение  │────▶│ Шифрование  │
│    файл     │     │  (padding)  │     │  на блоки   │     │   блоков    │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                                                   │
                                                          ┌────────┴────────┐
                                                          │                 │
                                                    ┌─────▼─────┐     ┌─────▼─────┐
                                                    │  Режим с  │     │ Режим без │
                                                    │    IV     │     │    IV     │
                                                    └─────┬─────┘     └─────┬─────┘
                                                          │                 │
                                                    ┌─────▼─────┐     ┌─────▼─────┐
                                                    │   XOR с   │     │    ECB    │
                                                    │    IV     │     │           │
                                                    └─────┬─────┘     └─────┬─────┘
                                                          │                 │
                                                    ┌─────▼─────────────────▼─────┐
                                                    │   10 раундов шифрования     │
                                                    │   (LSX преобразования)      │
                                                    └───────────────┬─────────────┘
                                                                    │
                                                    ┌───────────────▼─────────────┐
                                                    │   Добавление имитовставки   │
                                                    │   (если включена)           │
                                                    └───────────────┬─────────────┘
                                                                    │
                                                    ┌───────────────▼─────────────┐
                                                    │       Выходной файл         │
                                                    └─────────────────────────────┘

3.3 Детальная реализация криптографических преобразований
3.3.1 Умножение в поле Галуа GF(2^8)

Ядром линейных преобразований является умножение в конечном поле GF(2^8) с неприводимым полиномом x^8 + x^7 + x^6 + x + 1 (0xC3). Реализовано методом "столбика" (shift-and-add):
python

def _galois_multiply(self, a: int, b: int) -> int:
    """
    Умножение двух чисел в поле Галуа GF(2^8).
    
    Алгоритм: стандартное умножение в столбик с редуцированием
    по порождающему полиному 0xC3 при переполнении.
    
    Args:
        a: Первый множитель (0-255)
        b: Второй множитель (0-255)
        
    Returns:
        Результат умножения в поле GF(2^8)
    """
    p = 0
    counter = 0
    a_val = a & 0xFF
    b_val = b & 0xFF
    
    # Умножаем, пока не обработаем все 8 бит
    while counter < 8 and a_val != 0 and b_val != 0:
        # Если младший бит b установлен, добавляем a к результату
        if b_val & 1:
            p ^= a_val
        
        # Запоминаем старший бит a для редуцирования
        hi_bit = a_val & 0x80
        
        # Сдвигаем a влево
        a_val = (a_val << 1) & 0xFF
        
        # Если был установлен старший бит, выполняем редуцирование
        if hi_bit:
            a_val ^= self.POLY  # POLY = 0xC3
        
        # Сдвигаем b вправо
        b_val >>= 1
        counter += 1
    
    return p & 0xFF

3.3.2 Преобразование R (линейное перемешивание)

Преобразование R выполняет линейное перемешивание блока путем вычисления нового байта как линейной комбинации всех байтов с последующим сдвигом:
python

def _r_transformation(self, data: List[int]) -> List[int]:
    """
    Преобразование R (линейное перемешивание).
    
    R(a) = (a0 * L[0]) ⊕ (a1 * L[1]) ⊕ ... ⊕ (a15 * L[15]), a1, a2, ..., a15
    где умножение выполняется в поле Галуа.
    
    Args:
        data: Входной блок (16 байт)
        
    Returns:
        Блок после преобразования R
    """
    # Вычисляем новый байт как линейную комбинацию всех байтов
    a_zero = 0
    for i in range(16):
        a_zero ^= self._galois_multiply(data[i], self.L_VECTOR[i])
    
    # Сдвигаем все байты вправо и вставляем новый байт в начало
    state = [0] * 16
    for i in range(15, 0, -1):
        state[i] = data[i - 1]
    state[0] = a_zero
    
    return state

3.3.3 Линейное преобразование L

Линейное преобразование L представляет собой 16-кратное применение преобразования R:
python

def _l_transformation(self, data: List[int]) -> List[int]:
    """
    Линейное преобразование L.
    
    L = R^16 (16-кратное применение R)
    
    Args:
        data: Входной блок (16 байт)
        
    Returns:
        Блок после линейного преобразования
    """
    state = data.copy()
    for _ in range(16):
        state = self._r_transformation(state)
    return state

3.3.4 Нелинейное преобразование S

Нелинейное преобразование S реализует подстановку байтов с использованием таблицы замены (S-блока):
python

def _s_box_substitution(self, data: List[int]) -> List[int]:
    """
    Нелинейное преобразование S (подстановка).
    
    Каждый байт заменяется соответствующим значением из таблицы PI.
    
    Args:
        data: Входной блок (16 байт)
        
    Returns:
        Блок после подстановки
    """
    return [self.PI[b] for b in data]

Обратное преобразование использует обратную таблицу:
python

def _s_box_substitution_reverse(self, data: List[int]) -> List[int]:
    """
    Обратное нелинейное преобразование.
    
    Использует обратную таблицу REVERSE_PI.
    
    Args:
        data: Входной блок (16 байт)
        
    Returns:
        Блок после обратной подстановки
    """
    return [self.REVERSE_PI[b] for b in data]

3.3.5 Раундовое преобразование LSX

Один раунд шифрования объединяет наложение ключа (X), нелинейное преобразование (S) и линейное преобразование (L):
python

def _lsx_round(self, data: List[int], key: List[int]) -> List[int]:
    """
    Один раунд LSX преобразования.
    
    LSX(K) = L ∘ S ∘ X(K)
    
    Args:
        data: Входной блок
        key: Раундовый ключ
        
    Returns:
        Блок после раунда
    """
    # X - наложение ключа
    state = self._xor_lists(data, key)
    # Разворот (по спецификации ГОСТ)
    state = self._reverse(state)
    # S - нелинейное преобразование
    state = self._s_box_substitution(state)
    # L - линейное преобразование
    state = self._l_transformation(state)
    # Обратный разворот
    state = self._reverse(state)
    return state

3.3.6 Генерация раундовых ключей

Генерация 10 раундовых ключей из основного 32-байтового ключа выполняется через сеть Фейстеля:
python

def _expand_keys(self, key: bytes):
    """
    Развертка ключа - генерация 10 раундовых ключей из основного ключа.
    
    Алгоритм:
    1. Генерируются 32 итерационные константы
    2. Основной ключ делится на две половины A и B
    3. В 4 раундах по 8 итераций генерируются остальные ключи
    """
    key_list = list(key)
    
    # Шаг 1: Генерация 32 итерационных констант
    for i in range(32):
        iter_num = [0] * 16
        iter_num[15] = i + 1
        self.iter_c[i] = self._l_transformation(iter_num)
    
    # Шаг 2: Разделение ключа на две половины
    a = key_list[:16]   # Первая половина
    b = key_list[16:]   # Вторая половина
    
    # Первые два раундовых ключа
    self.iter_k[0] = b  # K1 = B
    self.iter_k[1] = a  # K2 = A
    
    # Шаг 3: Генерация остальных 8 ключей
    for i in range(4):          # 4 раунда
        for j in range(8):       # по 8 итераций
            idx = 8 * i + j
            # Преобразование Фейстеля
            tmp = self._feistel_network(a, b, self.iter_c[idx])
            a = tmp[1]
            b = tmp[0]
        
        # Сохраняем полученные ключи
        self.iter_k[2 * i + 2] = a
        self.iter_k[2 * i + 3] = b

3.3.7 Шифрование блока

Полное шифрование одного 16-байтового блока выполняется за 9 раундов LSX с финальным наложением ключа:
python

def encrypt_block(self, block: bytes) -> bytes:
    """
    Шифрование одного 16-байтового блока.
    
    Выполняется 9 раундов LSX с последующим финальным XOR с 10-м ключом.
    
    Args:
        block: 16-байтовый блок открытого текста
        
    Returns:
        16-байтовый блок шифртекста
    """
    if self.iter_k[0] is None:
        raise RuntimeError("Ключ не установлен")
    
    state = list(block)
    
    # 9 раундов шифрования
    for j in range(9):
        state = self._xor_lists(state, self.iter_k[j])
        state = self._s_box_substitution(state)
        state = self._l_transformation(state)
    
    # Финальный XOR с 10-м ключом
    state = self._xor_lists(state, self.iter_k[9])
    
    self.encrypt_count += 1
    return bytes(state)

3.3.8 Расшифрование блока

Расшифрование выполняется применением обратных преобразований в обратном порядке:
python

def decrypt_block(self, block: bytes) -> bytes:
    """
    Расшифрование одного 16-байтового блока.
    
    Выполняются обратные преобразования в обратном порядке.
    
    Args:
        block: 16-байтовый блок шифртекста
        
    Returns:
        16-байтовый блок открытого текста
    """
    if self.iter_k[0] is None:
        raise RuntimeError("Ключ не установлен")
    
    state = list(block)
    
    # Финальный XOR с 10-м ключом (в обратном порядке)
    state = self._xor_lists(state, self.iter_k[9])
    
    # 9 обратных раундов
    for j in range(8, -1, -1):
        state = self._l_transformation_reverse(state)
        state = self._s_box_substitution_reverse(state)
        state = self._xor_lists(state, self.iter_k[j])
    
    self.decrypt_count += 1
    return bytes(state)

3.4 Реализация режимов шифрования
3.4.1 Режим ECB (Electronic Codebook)
python

def encrypt_ecb(self, data: bytes, verbose: bool = False) -> bytes:
    """
    Режим простой замены (Electronic Codebook).
    
    Самый простой режим, где каждый блок шифруется независимо.
    Требует дополнения данных до размера блока.
    """
    padded = self._pad_data(data)
    result = bytearray()
    
    for i in range(0, len(padded), 16):
        block = padded[i:i+16]
        result.extend(self.encrypt_block(block))
    
    return bytes(result)

3.4.2 Режим CBC (Cipher Block Chaining)
python

def encrypt_cbc(self, data: bytes, iv: bytes, verbose: bool = False) -> bytes:
    """
    Режим сцепления блоков (Cipher Block Chaining).
    
    Каждый блок перед шифрованием XOR-ится с предыдущим зашифрованным блоком.
    Для первого блока используется вектор инициализации (IV).
    """
    padded = self._pad_data(data)
    result = bytearray()
    prev = iv
    
    for i in range(0, len(padded), 16):
        block = padded[i:i+16]
        # XOR с предыдущим зашифрованным блоком (или IV)
        xored = bytes([block[j] ^ prev[j] for j in range(16)])
        encrypted = self.encrypt_block(xored)
        result.extend(encrypted)
        prev = encrypted
    
    return bytes(result)

3.4.3 Режим CFB (Cipher Feedback)
python

def encrypt_cfb(self, data: bytes, iv: bytes, verbose: bool = False) -> bytes:
    """
    Режим обратной связи по шифртексту (Cipher Feedback).
    
    Превращает блочный шифр в поточный. Использует предыдущий шифртекст
    как вход для шифра.
    """
    result = bytearray()
    feedback = bytearray(iv)
    
    for i in range(0, len(data), 16):
        encrypted_feedback = self.encrypt_block(bytes(feedback))
        block = data[i:i+16]
        
        # XOR с зашифрованной обратной связью
        for j in range(len(block)):
            result.append(block[j] ^ encrypted_feedback[j])
        
        # Обновление обратной связи
        if len(block) == 16:
            feedback = result[-16:]
        else:
            for j in range(len(block)):
                feedback[j] = result[-len(block) + j]
    
    return bytes(result)

3.4.4 Режим OFB (Output Feedback)
python

def encrypt_ofb(self, data: bytes, iv: bytes, verbose: bool = False) -> bytes:
    """
    Режим обратной связи по выходу (Output Feedback).
    
    Превращает блочный шифр в синхронный поточный. Использует выход шифра
    как вход для следующего шага.
    """
    result = bytearray()
    state = bytearray(iv)
    
    for i in range(0, len(data), 16):
        encrypted_state = self.encrypt_block(bytes(state))
        state = bytearray(encrypted_state)
        
        block = data[i:i+16]
        for j in range(len(block)):
            result.append(block[j] ^ encrypted_state[j])
    
    return bytes(result)

3.4.5 Режим CTR (Counter)
python

def encrypt_ctr(self, data: bytes, iv: bytes, verbose: bool = False) -> bytes:
    """
    Режим счетчика (Counter).
    
    Превращает блочный шифр в поточный. Использует счетчик,
    который увеличивается для каждого блока.
    """
    result = bytearray()
    counter = bytearray(iv)
    
    for i in range(0, len(data), 16):
        encrypted_counter = self.encrypt_block(bytes(counter))
        
        block = data[i:i+16]
        for j in range(len(block)):
            result.append(block[j] ^ encrypted_counter[j])
        
        # Увеличение счетчика
        for j in range(15, -1, -1):
            counter[j] = (counter[j] + 1) & 0xFF
            if counter[j] != 0:
                break
    
    return bytes(result)

3.5 Реализация имитовставки (MAC)
3.5.1 Генерация подключей
python

def _generate_mac_keys(self) -> Tuple[bytearray, bytearray]:
    """
    Генерация подключей K1 и K2 для имитовставки.
    
    Returns:
        Кортеж (K1, K2) - подключи для MAC
    """
    # K1 = E(0)
    k1 = bytearray(16)
    k1 = bytearray(self.encrypt_block(bytes(k1)))
    
    # Сдвиг K1 влево
    carry = 0
    for i in range(15, -1, -1):
        new_carry = 1 if k1[i] >= 0x80 else 0
        k1[i] = ((k1[i] << 1) | carry) & 0xFF
        carry = new_carry
    if carry:
        k1[15] ^= 0x87
    
    # K2 = сдвиг K1
    k2 = bytearray(k1)
    carry = 0
    for i in range(15, -1, -1):
        new_carry = 1 if k2[i] >= 0x80 else 0
        k2[i] = ((k2[i] << 1) | carry) & 0xFF
        carry = new_carry
    if carry:
        k2[15] ^= 0x87
    
    return k1, k2

3.5.2 Вычисление имитовставки
python

def compute_mac(self, data: bytes, mac_len: int = 8, verbose: bool = False) -> bytes:
    """
    Вычисление имитовставки (MAC) для данных.
    
    Алгоритм:
    1. Генерация подключей K1 и K2
    2. Дополнение данных (0x80 + нули)
    3. CBC-подобное шифрование с использованием подключей
    """
    # Генерация подключей
    k1, k2 = self._generate_mac_keys()
    
    # Дополнение данных
    if len(data) % 16 == 0:
        padded = data
        is_added = False
    else:
        padded = data + b'\x80' + b'\x00' * (15 - (len(data) % 16))
        is_added = True
    
    # Разбиваем на блоки
    blocks = [padded[i:i+16] for i in range(0, len(padded), 16)]
    
    if not blocks:
        return bytes(mac_len)
    
    # Начальное состояние
    state = bytearray(self.encrypt_block(blocks[0]))
    
    # Промежуточные блоки
    for block in blocks[1:-1]:
        for j in range(16):
            state[j] ^= block[j]
        state = bytearray(self.encrypt_block(bytes(state)))
    
    # Последний блок с подключом
    last = blocks[-1]
    key = k2 if is_added else k1
    
    for j in range(16):
        state[j] ^= last[j] ^ key[j]
    
    state = bytearray(self.encrypt_block(bytes(state)))
    
    return bytes(state[:mac_len])

3.6 Дополнение данных (Padding)

В отличие от стандартного PKCS#7, в данной реализации используется специальная схема дополнения, аналогичная Kotlin версии:
python

def _pad_data(self, data: bytes) -> bytes:
    """
    Дополнение данных до размера блока по схеме из Kotlin.
    
    Схема дополнения:
    - Если данных не хватает до полного блока, добавляются нули
    - Последний байт устанавливается:
      * 0x80, если добавлен 1 байт
      * 0x81, если добавлено больше байт, и затем 0x01 перед последним нулем
    """
    if len(data) % 16 == 0:
        return data
    
    num_blocks = len(data) // 16 + 1
    number_of_null = num_blocks * 16 - len(data)
    padded = bytearray(data) + bytearray(number_of_null)
    
    if number_of_null == 1:
        padded[-1] = 0x80
    else:
        for i in range(len(padded) - 1, -1, -1):
            if i == len(padded) - 1:
                padded[i] = 0x81
            elif padded[i] != 0:
                padded[i + 1] = 0x01
                break
    
    return bytes(padded)

3.7 Генерация ключа из пароля

Для преобразования пользовательского пароля в 32-байтовый ключ используется 50 итераций SHA-512:
python

def _length_to_32_bytes(self, key_str: str) -> bytes:
    """
    Преобразование строки в 32-байтовый ключ.
    
    Используется 50 итераций SHA-512 для усиления стойкости.
    
    Args:
        key_str: Строка-пароль
        
    Returns:
        32-байтовый ключ
    """
    def sha512(data: bytes) -> bytes:
        return hashlib.sha512(data).digest()[:32]
    
    # Первая итерация SHA-512
    res = sha512(key_str.encode())
    
    # Еще 49 итераций (всего 50)
    for _ in range(50):
        res = sha512(res)
    
    return res

3.8 Управление ключами (класс KeyManager)

Класс KeyManager обеспечивает сохранение и загрузку ключей в директорию keys/:
python

class KeyManager:
    """Класс для управления ключами шифрования."""
    
    KEY_DIR = "keys"
    
    def __init__(self):
        if not os.path.exists(self.KEY_DIR):
            os.makedirs(self.KEY_DIR)
    
    def save_key(self, name: str, key: bytes) -> str:
        """Сохранение ключа в файл."""
        clean_name = "".join(c for c in name if c.isalnum() or c in "._- ")
        clean_name = clean_name.replace(" ", "_")
        
        filename = f"{clean_name}.key"
        filepath = os.path.join(self.KEY_DIR, filename)
        
        with open(filepath, 'wb') as f:
            f.write(key)
        
        return filepath
    
    def load_key(self, path: str) -> bytes:
        """Загрузка ключа из файла."""
        # Поиск в директории keys, если путь не абсолютный
        if not os.path.isabs(path) and not os.path.exists(path):
            test_path = os.path.join(self.KEY_DIR, path)
            if os.path.exists(test_path):
                path = test_path
        
        with open(path, 'rb') as f:
            data = f.read()
        
        # Поддержка ключей разного формата
        if len(data) == 32:
            return data
        elif len(data) == 48:
            print("⚠️ Обнаружен ключ старого формата (с солью). Используется только ключ.")
            return data[:32]
        else:
            raise ValueError(f"Неверный размер ключа: {len(data)} байт")

3.9 Интерактивный пользовательский интерфейс
3.9.1 Структура меню

Интерактивное меню реализует навигацию по номерам с подменю для различных операций:
python

class InteractiveMenu:
    def run(self):
        while True:
            self.clear()
            self.print_header()
            self.print_key_status()
            
            print("ГЛАВНОЕ МЕНЮ:")
            print("  [1] 🔐 Зашифровать файл")
            print("  [2] 🔓 Расшифровать файл")
            print("  [3] 📋 Выбрать режим шифрования")
            print("  [4] 🛡️  Настройки имитовставки (MAC)")
            print("  [5] 🔑 Управление ключами")
            print("  [6] 🎲 Установить синхропосылку (IV)")
            print("  [7] 📊 Статистика")
            print("  [8] 🧪 Тестовый цикл")
            print("  [0] 🚪 Выход")
            
            choice = input("Выберите действие: ").strip()
            # Обработка выбора...

3.9.2 Обработка имитовставки при шифровании
python

# Если включена имитовставка, вычисляем её и добавляем в конец файла
if self.use_mac:
    print(f"\n🛡️  Вычисление имитовставки...")
    mac = self.cipher.compute_mac(data, self.mac_len, verbose)
    print(f"   Имитовставка: {mac.hex()}")
    
    # Добавляем имитовставку в конец зашифрованных данных
    final_data = encrypted + mac
else:
    final_data = encrypted

3.9.3 Обработка имитовставки при расшифровании
python

# Если включена имитовставка, отделяем её от зашифрованных данных
if self.use_mac:
    encrypted = file_data[:-self.mac_len]
    expected_mac = file_data[-self.mac_len:]
    
    # ... расшифрование ...
    
    # Проверка имитовставки
    computed_mac = self.cipher.compute_mac(decrypted, self.mac_len, verbose)
    if computed_mac == expected_mac:
        print(f"   ✅ Имитовставка совпадает!")
    else:
        print(f"   ❌ Имитовставка НЕ совпадает!")
        # Опция сохранения при несовпадении

3.10 Тестирование программы
3.10.1 Тестовый цикл

Для автоматической проверки всех режимов реализован тестовый цикл:
python

def test_menu(self):
    """Тестовый цикл для проверки всех режимов."""
    test_data = b"Hello, World! This is a test message."
    test_key = os.urandom(32)
    self.cipher.set_key_from_bytes(test_key)
    
    for mode in ["ECB", "CBC", "CFB", "OFB", "CTR"]:
        # Генерация IV для режимов, где он нужен
        gamma = os.urandom(16) if mode in ["CBC", "CFB", "OFB", "CTR"] else None
        
        # Шифрование
        if mode == "ECB":
            encrypted = self.cipher.encrypt_ecb(test_data)
        # ... аналогично для других режимов
        
        # Расшифрование
        if mode == "ECB":
            decrypted = self.cipher.decrypt_ecb(encrypted)
        # ... аналогично
        
        # Проверка
        assert decrypted == test_data

3.10.2 Результаты тестирования

Тестовый цикл подтверждает корректную работу всех режимов:
text

ИТОГИ ТЕСТИРОВАНИЯ:
  ✅ ECB
  ✅ CBC
  ✅ CFB
  ✅ OFB
  ✅ CTR
  ✅ MAC

3.11 Особенности реализации
3.11.1 Преимущества

    Модульность: Четкое разделение на классы с инкапсуляцией функциональности

    Расширяемость: Легко добавить новые режимы шифрования

    Интерактивность: Удобный пользовательский интерфейс

    Отладка: Подробный вывод каждого шага при необходимости

    Безопасность: 50 итераций SHA-512 для генерации ключа из пароля

    Совместимость: Поддержка ключей разного формата (32/48 байт)

3.11.2 Ограничения

    Производительность: Python уступает в скорости компилируемым языкам

    Память: При работе с большими файлами ( > 1 ГБ) может потребоваться оптимизация

3.11.3 Возможные улучшения

    Добавление многопоточной обработки для больших файлов

    Реализация потоковой обработки (чтение/запись по частям)

    Добавление поддержки дополнительных режимов (XTS, GCM)

3.12 Инструкция по установке и запуску
3.12.1 Требования

    Python 3.10 или выше

    Стандартная библиотека (дополнительных зависимостей не требуется)

3.12.2 Запуск программы
bash

# Скачать файл программы
wget https://example.com/kuznechik.py

# Запустить
python kuznechik.py

3.12.3 Примеры использования

Шифрование файла:

    Установить ключ (пункт меню 5 → 1)

    Выбрать режим (пункт меню 3)

    Включить имитовставку при необходимости (пункт меню 4)

    Выбрать пункт 1 для шифрования

    Указать входной и выходной файлы

Расшифрование файла:

    Загрузить тот же ключ (пункт меню 5 → 2)

    Выбрать тот же режим (пункт меню 3)

    Убедиться, что настройки имитовставки совпадают

    Выбрать пункт 2 для расшифрования

    Указать зашифрованный файл и файл для результата
