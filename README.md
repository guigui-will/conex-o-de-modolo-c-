using System;
using System.IO;
using System.Net;
using System.Net.Sockets;
using System.Text;

class Program
{
    // Porta do servidor
    const int Port = 5000;

    // Pasta onde os arquivos serão salvos
    static readonly string OutputDir = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "Received");

    static void Main()
    {
        Directory.CreateDirectory(OutputDir);
        var listener = new TcpListener(IPAddress.Any, Port);
        listener.Start();
        Console.WriteLine($"[Servidor] Aguardando conexões na porta {Port}...");

        while (true)
        {
            using var client = listener.AcceptTcpClient();
            Console.WriteLine("[Servidor] Cliente conectado.");
            try
            {
                ReceiveFile(client);
                Console.WriteLine("[Servidor] Transferência concluída.\n");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"[Servidor] Erro: {ex.Message}");
            }
        }
    }

    static void ReceiveFile(TcpClient client)
    {
        using var network = client.GetStream();

        // 1) Ler metadados: nome e tamanho
        string fileName = ReadLine(network);
        string sizeStr  = ReadLine(network);

        if (!long.TryParse(sizeStr, out long fileSize))
            throw new InvalidDataException("Tamanho do arquivo inválido.");

        string destPath = Path.Combine(OutputDir, SanitizeFileName(fileName));
        Console.WriteLine($"[Servidor] Recebendo: {fileName} ({fileSize} bytes)");

        // 2) Receber conteúdo em blocos
        const int BufferSize = 64 * 1024;
        var buffer = new byte[BufferSize];
        long totalRead = 0;

        using var fs = new FileStream(destPath, FileMode.Create, FileAccess.Write, FileShare.None);
        int read;
        while (totalRead < fileSize && (read = network.Read(buffer, 0, (int)Math.Min(BufferSize, fileSize - totalRead))) > 0)
        {
            fs.Write(buffer, 0, read);
            totalRead += read;

            // Progresso simples
            if (fileSize > 0)
            {
                double pct = (double)totalRead / fileSize * 100.0;
                Console.Write($"\r[Servidor] {pct:0.0}%");
            }
        }
        Console.WriteLine();
    }

    static string ReadLine(NetworkStream stream)
    {
        var sb = new StringBuilder();
        int b;
        while ((b = stream.ReadByte()) != -1)
        {
            if (b == '\n') break;
            if (b != '\r') sb.Append((char)b);
        }
        return sb.ToString();
    }

    static string SanitizeFileName(string name)
    {
        foreach (var c in Path.GetInvalidFileNameChars())
            name = name.Replace(c, '_');
        return name;
    }
}# conex-o-de-modolo-c-
