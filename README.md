# dc
### :pushpin:Запуск сервера и ожидание клиента.
        
```C# 
        public void startServer()
        {
            if (!active)
            {
                listener.Bind(ipEndPoint);
                listener.Listen(port);

                udpClient.Client.Bind(ipEndPoint);

                active = true;
                Console.WriteLine("Сервер запущен. Ожидание подключений...");

                Task.Run(ReceiveRequest);
            }
            else
            {
                Console.WriteLine("Сервер уже запущен. Ожидание подключений...");
            }

        }
```

Клиент посылает запрос на подключение к серверу, происходит подключение. 
        
```C#
public async Task ConnectNewClient()
        {
            while (true)
            {
                clientSocket = await listener.AcceptAsync();
                Client client = new Client(clientSocket, this);
                Console.WriteLine($"Подключился игрок - {clientSocket.RemoteEndPoint}");

                try
                {
                    Task.Run(client.HandleResponseAsync);
                }
                catch (Exception ex)
                {
                    Console.WriteLine(ex.Message);
                    clientSocket.Shutdown(SocketShutdown.Both);
                    break;
                }
            }
            //clientSocket = listener.Accept();
            //Console.WriteLine($"Подключился игрок - {clientSocket.RemoteEndPoint}");
        }

```
### :pushpin:Начало игры
Сервер получает запрос клиента **“StartGame + fieldSize”**
(Сервер получает запрос клиента на начало игры и указывает, какое по размерам поле использовать: маленькое (20*10), среднее (20*15) или большое (20*20). 

SMALL,(20, 10)
MEDIUM(20, 15)
LARGE(20, 20)

Получив информацию, сервер устанавливает размеры поля в зависимости от запроса клиента. 

**тип данных (int rows, int cols)**

**На выход: GameStarted - Rows - Columns (размер поля)**

После установки размеров поля сервер отправляет клиенту сообщение о том, что игра началась, а также количество строк и колонок поля)
Сервер устанавливает  и отправляет клиенту размер поля, начальный блок, позицию текущего блока

.
```C# 
private async Task ReceiveRequest()
        {

            var clientEp = new IPEndPoint(IPAddress.Any, 0);
            var responseData = Encoding.ASCII.GetBytes("Come and play");

            while (true)
            {
                var receiveBUffer = udpClient.Receive(ref clientEp);

                if (Encoding.UTF8.GetString(receiveBUffer) == "Want to play")
                {
                    Console.WriteLine($"received request from {clientEp.Address}");
                    udpClient.Send(responseData, responseData.Length, clientEp.Address.ToString(), clientEp.Port);
                };
            }
        }
```
Если для фигуры на поле не хватает места, т.е. верхний ряд не пуст, она не размещается. Сервер посылает клиенту сообщение
об окончании игры, выводится сообщение об окончании  **await SendResponseAsync("GameOver")**.

```C# 
public void Stop()
        {
            listener.Shutdown(SocketShutdown.Both);
        }
```
Клиент запрашивает фигуру через сообщение - сервер передает информацию о конкретной фигуре - клиент получает инфу и рисует нужный блок. При запросе клиента на поворот фигуры или ее перемещение (клиент посылает сообщение с запросом),
сервер выполняет запрос. При запросе на перемещение за границы поля фигура не перемещается. Cервером обрабатываются следующие виды сообщений  
* "StartGame s/m/l"
* "NextFigure"
* "GetGrid"
* "GetBlock"
* "Left"
* "Right"
* "Rotate"
* "Drop"

### :pushpin:Запрос NextFigure

```C# 
CurrentBlock.BlockID = GetRandomBlock
```

Сервер устанавливает текущему блоку в поле **BlockID** значение ID случайного блока из списка существующих и передает это клиенту

### :pushpin:Запрос GetGrid

**На вход из запроса сервер получает параметры поля (int rows, int cols)**.
**На выход – обновленный счет (в зависимости от кол-ва строк) (string score), поле (int rows, int cols)**.

Если срабатывает функция **GameOver** из менеджера игры, **GetGrid** посылает клиенту сообщение об окончании игры. Иначе **GetGrid** проверяет игровое поле на наличие полностью заполненных строк и очищает их, вместе с этим добавляя к счету очки за заполнение. 

### :pushpin:Запрос GetBlock

**На выход: позиция текущего блока (int rows, int cols, int blockPosId)**
Хранит информацию о местоположении текущего блока 

### :pushpin:Запрос Left, Right:

```C#
BlockPicker.GetAndUpdate();
return block;
new Position(StartOffset.Row, StartOffset.Column)

```

**Возвращает позицию блока относительно ряда и колонны (int row, int column, int blockPosId)**

### :pushpin:Запрос Rotate

```C#
rotationState = (rotationState + 1) % BlockTiles.Length;

```
**возвращает rotationState (int)**

### :pushpin:Запрос Drop

возвращает позицию блока относительно ряда и колонны 
**(int row, int column, int blockPosId)**


### :pushpin:Виды блоков
```C# 
   
            IBlock (id=1),
            JBLock (id=2),
            LBlock (id=3),
            OBlock (id=4),
            SBlock (id=5),
            TBlock (id=6),
            ZBlock (id=7)
       

```

```C# 

```
