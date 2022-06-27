## SignalR - ChatApp | .NET 6

**Yangi Blazor Server Project ochamiz.**
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bdqd32iaobjqqnszw8xo.jpg)
```cs
dotnet new blazorserver -o ChatApp
```

**1- qadam** : Loyihaga _SignalR_ package ni qo'shamiz.

```cs
Microsoft.AspNetCore.SignalR.Client
```
**2- qadam** : _Hubs_ nomli Folder yaratib ichiga `ChatHub.cs` file ochamiz .

> Hubs/ChatHub.cs

```cs
using Microsoft.AspNetCore.SignalR;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace ChatApp.Hubs;
public class ChatHub : Hub
{
    public async Task SendMessage(string user, string message)
    {
        await Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}
```
**3- qadam** : `Program.cs` da ro'yxatdan o'tkazamiz.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r94z1hoo0xepr7ee8r65.jpg)

```cs
using ChatApp.Data;
using ChatApp.Hubs;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Web;
using Microsoft.AspNetCore.ResponseCompression;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();
builder.Services.AddSingleton<WeatherForecastService>();

builder.Services.AddResponseCompression(opts => {
    opts.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(new[] { "application/octet-stream" });
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();

app.UseStaticFiles();

app.UseRouting();

app.MapBlazorHub();

app.MapHub<ChatHub>("/chathub");

app.MapFallbackToPage("/_Host");

app.Run();
```
**4- qadam** : _Models_ Folder yaratib uni ichida `UserMessage.cs` ochamiz.

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace ChatApp.Models;
public class UserMessage
{
    public string Username { get; set; }
    public string Message { get; set; }
    public bool CurrentUser { get; set; }
    public DateTime DateSent { get; set; }
}
```
**5- qadam** : `Pages/Index.razor` ning ichidagilarni ochirib tashlaymiz va quyidagilarni yozamiz .

```cs
@page "/"
@using Microsoft.AspNetCore.SignalR.Client
@using Models
@inject NavigationManager NavigationManager
@implements IAsyncDisposable

<div class="container overflow-auto shadow-sm p-3 mb-5 bg-white rounded" style="height: 500px;">
    @if (!userMessages.Any())
    {
        <p>No messages yet, start chatting!</p>
    }

    @foreach (var userMessage in userMessages)
    {
        <div class="row mb-3 d-flex @(userMessage.CurrentUser ? "justify-content-end" : "")">
            <div class="card shadow @(userMessage.CurrentUser ? "p-3 mb-2 bg-primary text-white" : "ml-5")" style="width: 18rem;">
                <div class="card-header">
                    @(userMessage.CurrentUser ? "You" : userMessage.Username)
                </div>
                <ul class="list-group list-group-flush">
                    <li class="list-group-item @(userMessage.CurrentUser ? "p-3 mb-2 bg-primary text-white" : "")">@userMessage.Message</li>
                </ul>
                <div class="card-footer">
                    <span class="small">@userMessage.DateSent.ToString("HH:mm | MMM dd")</span>
                </div>
            </div>
        </div>
    }
</div>

<div class="container">
    <div class="row">
        <div class="col-3">
            <input @bind="usernameInput" type="text" class="form-control" placeholder="Your name" readonly="@isUserReadonly"/>
        </div>
        <div class="col-6">
            <textarea @bind="messageInput" class="form-control" placeholder="Start typing..."></textarea>
        </div>
        <div class="col-3">
            <button type="button" @onclick="Send" disabled="@(!IsConnected)" class="btn btn-primary">Send</button>
        </div>
    </div>
</div>

@code{
    private HubConnection hubConnection;
    private List<UserMessage> userMessages = new();
    private string usernameInput;
    private string messageInput;
    private bool isUserReadonly = false;

    public bool IsConnected => hubConnection.State == HubConnectionState.Connected;

    protected override async Task OnInitializedAsync()
    {
        hubConnection = new HubConnectionBuilder()
            .WithUrl(NavigationManager.ToAbsoluteUri("/chathub"))
            .Build();

        hubConnection.On<string, string>("ReceiveMessage", (user, message) =>
        {
            userMessages.Add(new UserMessage { Username = user, Message = message, CurrentUser = user == usernameInput, DateSent = DateTime.Now });

            StateHasChanged();
        });

        await hubConnection.StartAsync();
    }

    private async Task Send()
    {
        if (!string.IsNullOrEmpty(usernameInput) && !string.IsNullOrEmpty(messageInput))
        {
            await hubConnection.SendAsync("SendMessage", usernameInput, messageInput);

            isUserReadonly = true;
            messageInput = string.Empty;
        }
    }

    public async ValueTask DisposeAsync()
    {
        if (hubConnection is not null)
        {
            await hubConnection.DisposeAsync();
        }
    }
}
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hr4skx6sh2usjnf5i8ne.png)

[Source Code](https://github.com/Xakimov1610/SignalR-ChatApp-Simple)