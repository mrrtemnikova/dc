# dc
 Запуск сервера и ожидание клиента.
        
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
Для начала игры сервер должен получить сообщение от клиента "Want to play" и отправить в ответ "Come and play".
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
об окончании игры, выводится сообщение об окончании  await SendResponseAsync("GameOver").

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

```C# 
public async Task HandleResponseAsync()
        {
            try
            {
                while (true)
                {
                    var buffer = new List<byte>();
                    var bytesRead = new byte[1];

                    while (true)
                    {

                        var nextByte = await clientSocket.ReceiveAsync(bytesRead, SocketFlags.None);

                        if (nextByte == 0 || bytesRead[0] == '\n') break;

                        buffer.Add(bytesRead[0]);

                    }

                    var response = Encoding.UTF8.GetString(buffer.ToArray());

                    switch (response.Split(' ')[0])
                    {
                        case "StartGame":
                            var fieldSize = response.Split(' ')[1];
                            switch (fieldSize)
                            {
                                case "s":
                                    gameManager = new GameManager(server.FieldSizes[FieldSize.SMALL].Rows, server.FieldSizes[FieldSize.SMALL].Columns);
                                    break;
                                case "m":
                                    gameManager = new GameManager(server.FieldSizes[FieldSize.MEDIUM].Rows, server.FieldSizes[FieldSize.MEDIUM].Columns);
                                    break;
                                case "l":
                                    gameManager = new GameManager(server.FieldSizes[FieldSize.LARGE].Rows, server.FieldSizes[FieldSize.LARGE].Columns);
                                    break;
                            }
                            await SendResponseAsync($"GameStarted-{gameManager.Field.Rows}-{gameManager.Field.Columns}");
                            Console.WriteLine("Игра началась");
                            break;
                        case "NextFigure":
                            await SendResponseAsync(gameManager.CurrentBlock.BlockId.ToString());
                            break;
                        case "GetGrid":
                            StringBuilder stringBuilder = new StringBuilder();
                            if (gameManager.GameOver)
                            {
                                await SendResponseAsync("GameOver");
                                break;
                            }

                            for (int i = 0; i < gameManager.Field.Rows; i++)
                            {
                                for (int j = 0; j < gameManager.Field.Columns; j++)
                                {
                                    stringBuilder.Append(gameManager.Field[i, j]);
                                    stringBuilder.Append('-');
                                }
                                stringBuilder.Append('n');
                            }

                            stringBuilder.Append(gameManager.Score);
                            await SendResponseAsync(stringBuilder.ToString());
                            break;
                        case "GetBlock":
                            string positions = "";
                            foreach (var position in gameManager.CurrentBlock.BlockTilesPositions())
                            {
                                positions += position.Row + "-" + position.Column + "-" + position.BlockPosId + "n";
                            }
                            //positions += gameManager.CurrentBlock.BlockId;
                            await SendResponseAsync(positions);
                            break;
                        case "Left":
                            gameManager.MoveBlockLeft();
                            break;
                        case "Right":
                            gameManager.MoveBlockRight();
                            break;
                        case "Rotate":
                            gameManager.RotateBlock();
                            break;
                        case "Drop":
                            gameManager.DropBlock();
                            break;
                    }
                }

            }
            catch (Exception e)
            {
                throw new Exception(e.Message);
            }
        }

```

```C# 

```

```C# 

```
