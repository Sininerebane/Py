Домашнее задание: Свойства хорошего кода
Описание проблем и предложений по исправлению/улучшению приложите отдельным файлом/документом.

1) DEF DEPOSIT(SELF, AMOUNT: FLOAT):
    SELF.BANK.ACCOUNTS[SELF.ACCOUNT_NUMBER].BALANCE += AMOUNT

   Нарушение инкапсуляции: Метод deposit напрямую обращается к атрибуту accounts объекта bank, который является словарем счетов. Затем он изменяет атрибут balance конкретного счета, который также принадлежит другому объекту (инстансу Account). Такой подход нарушает принцип инкапсуляции в объектно-ориентированном программировании, где каждый объект должен управлять своими данными самостоятельно.    

    Прямой доступ к внутреннему состоянию другого объекта: Метод deposit обращается к словарю accounts в объекте bank. Это словарь, где каждый ключ — это номер счета, а значение — это объект Account. Затем метод изменяет значение balance прямо в этом объекте. Такой подход нарушает инкапсуляцию, потому что объект Account должен сам управлять своим балансом, а не позволять другим объектам напрямую изменять его.

    Отсутствие контроля над изменениями: Изменение баланса происходит без каких-либо проверок или валидации внутри метода deposit. Это означает, что можно передать отрицательное значение, что фактически будет считаться снятием денег со счета через метод пополнения. Кроме того, если бы были какие-то дополнительные правила или ограничения на операции с балансом (например, максимальный ежедневный лимит пополнения), их было бы сложно внедрить, поскольку данные изменяются напрямую без какого-либо посредничества.
 
 
 2) SELF.BANK = BANK В КЛАССЕ ACCOUNT
 
    Высокая связанность: Класс Account напрямую зависел от класса Bank, включая его внутреннюю структуру и методы. Это могло привести к сложностям при изменении реализации класса Bank, так как любые изменения в Bank могли требовать изменений в Account.

    Нарушение инкапсуляции: Используя self.bank для доступа к аккаунтам и управлению ими из класса Account, инкапсуляция была нарушена. Аккаунт не должен напрямую изменять данные в Bank или других аккаунтах.

    Упрощение тестирования: Удаление прямой связи между Account и Bank упрощает тестирование этих классов, так как они становятся менее зависимы друг от друга. Тесты для Account могут быть проведены без необходимости создания и настройки объекта Bank.
    
    
 Изменения:
 
	В улучшенном варианте кода предполагается, что вся логика управления аккаунтами (создание, поиск, изменение баланса) должна быть централизованно управляема через класс Bank. Таким образом, класс Account становится более независимым и ответственным только за управление своим собственным состоянием (например, балансом), а все операции связанные с аккаунтами выполняются через класс Bank:

    Класс Bank: управляет всеми аккаунтами, создает новые аккаунты и предоставляет доступ к ним.
    Класс Account: управляет своим балансом и другой информацией, касающейся только этого аккаунта.

	Эта модификация улучшает модульность и расширяемость системы, делая ее более устойчивой к изменениям и легче поддерживаемой. Если в вашей бизнес-логике необходимо, чтобы аккаунт напрямую взаимодействовал с банком, то можно рассмотреть возможность введения интерфейсов или сервисов, которые могут управлять этими взаимодействиями на более высоком уровне абстракции.
	

3) def create_account(self, account_number: int, customer_name: str, customer_address: str, initial_balance: float) -> Account:
    account = Account(self, account_number, customer_name, customer_address, initial_balance)
    self.accounts[account_number] = account
    return account
    
    Изменения:
    
    Удаление ссылки на Bank в Account: В новом методе объект Account создается без передачи в него экземпляра Bank. Это уменьшает связанность между Account и Bank, делая класс Account более независимым и легче тестируемым. Счет теперь не знает о существовании банка, что улучшает инкапсуляцию и модульность.
    Генерация номера счета внутри Bank: Вместо того, чтобы требовать внешнюю передачу номера счета, Bank сам генерирует уникальный номер счета. Это упрощает создание счетов и предотвращает возможные конфликты с номерами счетов, которые могли возникнуть при внешнем управлении номерами.
    Управление счетами все еще в Bank: Несмотря на изменения, управление счетами (хранение и доступ) все еще осуществляется через Bank, что поддерживает контроль и безопасность операций с счетами в банке.
    
    
 4)def open_account(self, initial_balance: float) -> Account:
    account_number = self._generate_account_number()
    account = self.bank.create_account(account_number, self.name, self.address, initial_balance)
    return account
    

    Генерация номера счета внутри Customer: Вызов self._generate_account_number() в классе Customer подразумевает, что клиент самостоятельно генерирует номер счета. Это может создать проблему, так как управление уникальностью номеров счетов должно быть централизовано в классе Bank для избежания конфликтов и коллизий номеров счетов. Лучше позволить банку управлять всеми номерами счетов.
    
    Изменения:

    Customer не занимается генерацией номера счета. Вместо этого Bank полностью контролирует создание счетов, включая генерацию уникального номера. Это уменьшает связанность между классами и улучшает инкапсуляцию, делая систему более модульной и устойчивой к ошибкам.








1) Низкий уровень абстракции

Абстракция в ООП означает способность класса представлять сущность с точки зрения того, какими действиями и характеристиками она обладает, скрывая при этом детали реализации.

    Пример: В вашем коде методы deposit и withdraw в классе Account обращаются напрямую к атрибутам другого объекта (self.bank.accounts[self.account_number].balance). Такой подход раскрывает внутреннее устройство объекта Bank и связанных с ним аккаунтов, что снижает уровень абстракции. Каждый класс должен управлять своими данными и поведением без прямого доступа к внутреннему состоянию других объектов.

2) Высокая связность

Связность относится к тому, насколько сильно связаны компоненты программы. Высокая связность обычно не желательна, так как делает систему менее модульной и затрудняет изменение и тестирование ее частей независимо друг от друга.

    Пример: В методе deposit, использование self.bank.accounts[self.account_number].balance += amount показывает, что класс Account напрямую зависит от внутренней структуры Bank. Изменения в реализации Bank потребуют соответствующих изменений в Account, что является показателем высокой связности.

3) Низкая целостность

Целостность в контексте ООП связана с тем, насколько надежно данные защищены от некорректного использования, и насколько четко отделены ответственности между различными частями программы.

    Пример: Метод deposit не только выполняет операции, которые должны быть инкапсулированы в Account (например, проверка на отрицательные суммы), но и изменяет состояние другого объекта. Это может привести к ошибкам, если, например, одновременно из разных мест программы будут выполнены операции с балансом.

4) Проблемы с компоновкой

Компоновка связана с тем, как компоненты системы разделены и как они взаимодействуют. Неправильная компоновка может привести к сложностям в расширении системы, добавлении новых функций или в поддержке.

    Пример: Генерация номера счета в классе Customer может привести к дублированию номеров, особенно если банк имеет много клиентов и аккаунтов. В идеале, управление номерами счетов должно быть централизованно в Bank, чтобы обеспечить их уникальность и целостность данных.

Ваш код демонстрирует типичные проблемы, которые могут возникнуть при разработке объектно-ориентированных систем без четкого соблюдения принципов разделения ответственностей и инкапсуляции. Пересмотр этих аспектов может значительно улучшить дизайн системы, ее надежность, тестируемость и расширяемость.
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
