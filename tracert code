using System;
using System.Diagnostics;
using System.Net;
using System.Net.Sockets;


class TracerouteUtility
{
    static void Main(string[] args)
    {
        while (true)
        {
            Console.Write("Введите команду (tracert IP/DNS): ");
            string input = Console.ReadLine();

            if (string.IsNullOrWhiteSpace(input))
                continue;

            string[] parts = input.Split(new[] { ' ' }, StringSplitOptions.RemoveEmptyEntries);

            if (parts.Length != 2 || parts[0].ToLower() != "tracert")
            {
                Console.WriteLine("Ошибка: Используйте команду в формате tracert <IP>");
                continue;
            }

            string target = parts[1];

            try
            {
                TraceRoute(target);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Ошибка: {ex.Message}");
            }

            Console.WriteLine("\nТрассировка завершена.\n");
        }
    }

    static void TraceRoute(string target)
    {
        IPAddress targetAddress = ResolveTargetAddress(target);
        Console.WriteLine($"Трассировка маршрута к {target} [{targetAddress}] с максимальным числом прыжков 30:\n");

        using (Socket socket = new Socket(AddressFamily.InterNetwork, SocketType.Raw, ProtocolType.Icmp))
        {
            socket.SetSocketOption(SocketOptionLevel.Socket, SocketOptionName.ReceiveTimeout, 3000);

            for (int ttl = 1; ttl <= 30; ttl++)
            {
                socket.SetSocketOption(SocketOptionLevel.IP, SocketOptionName.IpTimeToLive, ttl);
                EndPoint remoteEndPoint = new IPEndPoint(IPAddress.Any, 0);

                long[] responseTimes = new long[3];
                IPAddress hopAddress = null;

                for (int i = 0; i < 3; i++)
                {
                    byte[] packet = BuildIcmpPacket((ushort)(ttl + i));
                    Stopwatch timer = Stopwatch.StartNew();

                    try //доп задание
                    {
                        socket.SendTo(packet, new IPEndPoint(targetAddress, 0));
                        byte[] buffer = new byte[1024];
                        int bytesReceived = socket.ReceiveFrom(buffer, ref remoteEndPoint);
                        timer.Stop();

                        responseTimes[i] = timer.ElapsedMilliseconds;
                        hopAddress = ((IPEndPoint)remoteEndPoint).Address;
                    }
                    catch (SocketException)
                    {
                        responseTimes[i] = -1;
                    }
                    Thread.Sleep(100);
                }

                DisplayHopInfo(ttl, responseTimes, hopAddress);

                if (hopAddress != null && hopAddress.Equals(targetAddress))
                    break;
            }
        }
    }

    static IPAddress ResolveTargetAddress(string target)
    {
        try
        {
            // Используем Dns.GetHostEntry для получения информации о хосте
            IPHostEntry hostEntry = Dns.GetHostEntry(target);

            // Ищем первый адрес в списке
            foreach (IPAddress address in hostEntry.AddressList)
            {
                if (address.AddressFamily == AddressFamily.InterNetwork)
                {
                    return address;
                }
            }

            return hostEntry.AddressList[0];
        }
        // Если адрес не найден
        catch (Exception)
        {
            throw new Exception("адрес не найден.");
        }
    }

    static void DisplayHopInfo(int ttl, long[] times, IPAddress hopAddress)
    {
        string time1 = times[0] >= 0 ? $"{times[0]} ms" : "*";
        string time2 = times[1] >= 0 ? $"{times[1]} ms" : "*";
        string time3 = times[2] >= 0 ? $"{times[2]} ms" : "*";
        if (hopAddress == null)
        {
            Console.WriteLine($"{ttl,3}   {time1,5}   {time2,5}   {time3,5}   Превышен интервал ожидания для запроса.");
        }
        else
        {
            string hostName = GetHostName(hopAddress);
            Console.WriteLine($"{ttl,3}   {time1,5}   {time2,5}   {time3,5}   {hostName} [{hopAddress}]");
        }
    }

    static string GetHostName(IPAddress ip) //доп задание прод.
    {
        try
        {
            return Dns.GetHostEntry(ip).HostName;
        }
        catch
        {
            return ip.ToString();
        }
    }

    static byte[] BuildIcmpPacket(ushort sequenceNumber)
    {
        // Заголовок ICMP + данныx
        byte[] packet = new byte[72];

        // Тип и код эхо реквеста
        packet[0] = 8;
        packet[1] = 0;
        packet[2] = 0;
        packet[3] = 0;
        packet[4] = 0;
        packet[5] = 1;
        packet[6] = (byte)(sequenceNumber >> 8);
        packet[7] = (byte)(sequenceNumber & 0xFF);

        // Данные (64 байта, заполнены нулями)
        for (int i = 8; i < packet.Length; i++)
        {
            packet[i] = 0;
        }

        // Вычисляем контрольную сумму
        ushort checksum = CalculateChecksum(packet);
        packet[2] = (byte)(checksum >> 8);
        packet[3] = (byte)(checksum & 0xFF);

        return packet;
    }

    static ushort CalculateChecksum(byte[] data)
    {
        uint sum = 0;
        for (int i = 0; i < data.Length; i += 2)
        {
            ushort word = (ushort)((data[i] << 8) + (i + 1 < data.Length ? data[i + 1] : 0));
            sum += word;
        }

        while ((sum >> 16) != 0)
        {
            sum = (sum & 0xFFFF) + (sum >> 16);
        }
        return (ushort)~sum;
    }
}
