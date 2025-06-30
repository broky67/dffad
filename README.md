using Serilog;

var builder = Host.CreateDefaultBuilder(args)
    .UseSerilog((ctx, config) =>
    {
        config.WriteTo.File(
            path: "Logs/service.log",
            rollingInterval: RollingInterval.Day,
            retainedFileCountLimit: 7);
    })
    .ConfigureServices(services =>
    {
        services.AddHostedService<Worker>();
    });

await builder.Build().RunAsync();
