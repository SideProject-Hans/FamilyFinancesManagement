---
mode: 'agent'
tools: ['codebase', 'editFiles', 'terminalCommand']
description: 'Setup Blazor project with Radzen UI components including package installation and configuration'
---

# Blazor Radzen Setup

Your goal is to help me setup a Blazor project with Radzen UI components, including package installation, configuration, and basic implementation examples.

## Project Detection

First, detect the Blazor project type:
- Blazor Server App
- Blazor WebAssembly App
- Blazor Hybrid (MAUI)
- Blazor Auto (Interactive Server and WebAssembly)

## Package Installation

Install the appropriate Radzen packages:

### Core Radzen Packages
```bash
# Core Radzen Blazor Components
dotnet add package Radzen.Blazor

# Optional: Radzen.Blazor.Data for data grid functionality
dotnet add package Radzen.Blazor.Data
```

### Additional Packages (if needed)
```bash
# For Entity Framework integration
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools

# For HTTP client operations
dotnet add package System.Net.Http.Json
```

## Configuration Steps

### 1. Program.cs Configuration

Add Radzen services to the dependency injection container:

**For Blazor Server:**
```csharp
using Radzen;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();

// Add Radzen services
builder.Services.AddScoped<DialogService>();
builder.Services.AddScoped<NotificationService>();
builder.Services.AddScoped<TooltipService>();
builder.Services.AddScoped<ContextMenuService>();

var app = builder.Build();

// Configure the HTTP request pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.MapRazorPages();
app.MapBlazorHub();
app.MapFallbackToPage("/_Host");

app.Run();
```

**For Blazor WebAssembly:**
```csharp
using Microsoft.AspNetCore.Components.Web;
using Microsoft.AspNetCore.Components.WebAssembly.Hosting;
using Radzen;

var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");

builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });

// Add Radzen services
builder.Services.AddScoped<DialogService>();
builder.Services.AddScoped<NotificationService>();
builder.Services.AddScoped<TooltipService>();
builder.Services.AddScoped<ContextMenuService>();

await builder.Build().RunAsync();
```

### 2. _Imports.razor Updates

Add Radzen namespaces to `_Imports.razor`:

```razor
@using System.Net.Http
@using System.Net.Http.Json
@using Microsoft.AspNetCore.Components.Forms
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.Web
@using Microsoft.AspNetCore.Components.Web.Virtualization
@using Microsoft.AspNetCore.Components.WebAssembly.Http
@using Microsoft.JSInterop
@using Radzen
@using Radzen.Blazor
```

### 3. CSS and JavaScript References

Update the main layout file (e.g., `MainLayout.razor`, `App.razor`, or `_Host.cshtml`):

**Add to the `<head>` section:**
```html
<!-- Radzen CSS -->
<link rel="stylesheet" href="_content/Radzen.Blazor/css/material-base.css">
<!-- Alternative themes:
<link rel="stylesheet" href="_content/Radzen.Blazor/css/bootstrap-base.css">
<link rel="stylesheet" href="_content/Radzen.Blazor/css/fluent-base.css">
<link rel="stylesheet" href="_content/Radzen.Blazor/css/humanistic-base.css">
<link rel="stylesheet" href="_content/Radzen.Blazor/css/software-base.css">
<link rel="stylesheet" href="_content/Radzen.Blazor/css/standard-base.css">
-->
```

**Add before closing `</body>` tag:**
```html
<!-- Radzen JavaScript -->
<script src="_content/Radzen.Blazor/Radzen.Blazor.js"></script>
```

### 4. Layout Component Updates

Update your main layout to include Radzen dialog and notification containers:

```razor
@inherits LayoutView
@inject IJSRuntime JSRuntime

<div class="page">
    <div class="sidebar">
        <NavMenu />
    </div>

    <main>
        <div class="top-row px-4">
            <a href="https://docs.microsoft.com/aspnet/" target="_blank">About</a>
        </div>

        <article class="content px-4">
            @Body
        </article>
    </main>
</div>

<!-- Radzen Components -->
<RadzenDialog />
<RadzenNotification />
<RadzenContextMenu />
<RadzenTooltip />
```

## Basic Implementation Examples

### Example 1: Data Grid with CRUD Operations

```razor
@page "/products"
@inject DialogService DialogService
@inject NotificationService NotificationService

<PageTitle>Products</PageTitle>

<h1>Products Management</h1>

<RadzenDataGrid @ref="productsGrid"
                Data="@products"
                TItem="Product"
                AllowFiltering="true"
                AllowPaging="true"
                AllowSorting="true"
                EditMode="DataGridEditMode.Single"
                RowUpdate="@OnUpdateRow"
                RowCreate="@OnCreateRow"
                Sort="@Reset"
                Page="@Reset"
                Filter="@Reset">
    <Columns>
        <RadzenDataGridColumn TItem="Product" Property="Id" Title="ID" Width="60px" />
        <RadzenDataGridColumn TItem="Product" Property="Name" Title="Name">
            <EditTemplate Context="product">
                <RadzenTextBox @bind-Value="product.Name" Style="width:100%; display: block" Name="Name" />
                <RadzenRequiredValidator Text="Name is required" Component="Name" Popup="true" />
            </EditTemplate>
        </RadzenDataGridColumn>
        <RadzenDataGridColumn TItem="Product" Property="Price" Title="Price">
            <Template Context="product">
                @String.Format(new System.Globalization.CultureInfo("en-US"), "{0:C}", product.Price)
            </Template>
            <EditTemplate Context="product">
                <RadzenNumeric @bind-Value="product.Price" Style="width:100%" />
            </EditTemplate>
        </RadzenDataGridColumn>
        <RadzenDataGridColumn TItem="Product" Context="product" Filterable="false" Sortable="false" TextAlign="TextAlign.Right" Width="156px">
            <Template Context="product">
                <RadzenButton Icon="edit" ButtonStyle="ButtonStyle.Light" Variant="Variant.Flat" Size="Size.Medium" Click="@(args => EditRow(product))" @onclick:stopPropagation="true">
                </RadzenButton>
                <RadzenButton ButtonStyle="ButtonStyle.Danger" Icon="delete" Variant="Variant.Flat" Shade="Shade.Lighter" Size="Size.Medium" class="my-1 ms-1" Click="@(args => DeleteRow(product))" @onclick:stopPropagation="true">
                </RadzenButton>
            </Template>
            <EditTemplate Context="product">
                <RadzenButton Icon="check" ButtonStyle="ButtonStyle.Success" Variant="Variant.Flat" Size="Size.Medium" Click="@((args) => SaveRow(product))">
                </RadzenButton>
                <RadzenButton Icon="close" ButtonStyle="ButtonStyle.Light" Variant="Variant.Flat" Size="Size.Medium" class="my-1 ms-1" Click="@((args) => CancelEdit(product))">
                </RadzenButton>
                <RadzenButton ButtonStyle="ButtonStyle.Danger" Icon="delete" Variant="Variant.Flat" Shade="Shade.Lighter" Size="Size.Medium" class="my-1 ms-1" Click="@(args => DeleteRow(product))">
                </RadzenButton>
            </EditTemplate>
        </RadzenDataGridColumn>
    </Columns>
</RadzenDataGrid>

<RadzenButton ButtonStyle="ButtonStyle.Success" Icon="add_circle_outline" class="mt-2 mb-4" Text="Add New Product" Click="@InsertRow" />

@code {
    RadzenDataGrid<Product> productsGrid;
    List<Product> products = new List<Product>();

    protected override async Task OnInitializedAsync()
    {
        await base.OnInitializedAsync();

        // Sample data
        products = new List<Product>
        {
            new Product { Id = 1, Name = "Product 1", Price = 10.99m },
            new Product { Id = 2, Name = "Product 2", Price = 20.99m },
            new Product { Id = 3, Name = "Product 3", Price = 30.99m }
        };
    }

    async Task EditRow(Product product)
    {
        await productsGrid.EditRow(product);
    }

    async Task OnUpdateRow(Product product)
    {
        Reset();
        NotificationService.Notify(NotificationSeverity.Success, "Product Updated", "Product has been updated successfully.");
    }

    async Task OnCreateRow(Product product)
    {
        products.Add(product);
        NotificationService.Notify(NotificationSeverity.Success, "Product Created", "New product has been created successfully.");
    }

    async Task SaveRow(Product product)
    {
        await productsGrid.UpdateRow(product);
    }

    void CancelEdit(Product product)
    {
        Reset();
        productsGrid.CancelEditRow(product);
    }

    async Task DeleteRow(Product product)
    {
        var result = await DialogService.Confirm("Are you sure you want to delete this product?", "Delete Product", new ConfirmOptions() { OkButtonText = "Yes", CancelButtonText = "No" });
        if (result.HasValue && result.Value)
        {
            products.Remove(product);
            await productsGrid.Reload();
            NotificationService.Notify(NotificationSeverity.Success, "Product Deleted", "Product has been deleted successfully.");
        }
    }

    async Task InsertRow()
    {
        var newProduct = new Product { Id = products.Count + 1 };
        await productsGrid.InsertRow(newProduct);
    }

    void Reset()
    {
        // Optional: Reset any filters or state
    }

    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public decimal Price { get; set; }
    }
}
```

### Example 2: Form with Validation

```razor
@page "/contact"
@inject NotificationService NotificationService

<PageTitle>Contact Form</PageTitle>

<h1>Contact Us</h1>

<RadzenTemplateForm TItem="ContactForm" Data="@model" Submit="@OnSubmit">
    <RadzenFieldset Text="Contact Information">
        <div class="row">
            <div class="col-md-6">
                <RadzenFormField Text="First Name" Variant="Variant.Outlined">
                    <RadzenTextBox @bind-Value="model.FirstName" Name="FirstName" />
                    <RadzenRequiredValidator Text="First name is required" Component="FirstName" />
                </RadzenFormField>
            </div>
            <div class="col-md-6">
                <RadzenFormField Text="Last Name" Variant="Variant.Outlined">
                    <RadzenTextBox @bind-Value="model.LastName" Name="LastName" />
                    <RadzenRequiredValidator Text="Last name is required" Component="LastName" />
                </RadzenFormField>
            </div>
        </div>

        <div class="row">
            <div class="col-md-12">
                <RadzenFormField Text="Email" Variant="Variant.Outlined">
                    <RadzenTextBox @bind-Value="model.Email" Name="Email" />
                    <RadzenRequiredValidator Text="Email is required" Component="Email" />
                    <RadzenEmailValidator Text="Please enter a valid email" Component="Email" />
                </RadzenFormField>
            </div>
        </div>

        <div class="row">
            <div class="col-md-12">
                <RadzenFormField Text="Message" Variant="Variant.Outlined">
                    <RadzenTextArea @bind-Value="model.Message" Name="Message" Rows="5" />
                    <RadzenRequiredValidator Text="Message is required" Component="Message" />
                </RadzenFormField>
            </div>
        </div>

        <div class="row">
            <div class="col-md-12">
                <RadzenButton ButtonType="ButtonType.Submit"
                             Size="Size.Large"
                             Icon="send"
                             Text="Send Message"
                             ButtonStyle="ButtonStyle.Primary" />
            </div>
        </div>
    </RadzenFieldset>
</RadzenTemplateForm>

@code {
    ContactForm model = new ContactForm();

    void OnSubmit()
    {
        NotificationService.Notify(NotificationSeverity.Success, "Form Submitted", "Your message has been sent successfully!");
        model = new ContactForm(); // Reset form
    }

    public class ContactForm
    {
        public string FirstName { get; set; } = string.Empty;
        public string LastName { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public string Message { get; set; } = string.Empty;
    }
}
```

## Theme Customization

To customize the Radzen theme, you can:

1. Choose from predefined themes by changing the CSS reference
2. Create custom CSS to override specific styles
3. Use CSS variables for theme customization

### Custom Theme Example

```css
/* wwwroot/css/custom-radzen.css */
:root {
    --rz-primary: #007bff;
    --rz-secondary: #6c757d;
    --rz-success: #28a745;
    --rz-info: #17a2b8;
    --rz-warning: #ffc107;
    --rz-danger: #dc3545;
    --rz-light: #f8f9fa;
    --rz-dark: #343a40;
}

.rz-button-primary {
    background-color: var(--rz-primary);
    border-color: var(--rz-primary);
}
```

## Advanced Features

### Data Virtualization for Large Datasets

```razor
<RadzenDataGrid Data="@largeDataset"
                TItem="DataItem"
                AllowVirtualization="true"
                Style="height: 400px"
                LoadData="@LoadData">
    <!-- Columns definition -->
</RadzenDataGrid>
```

### Chart Integration

```razor
<RadzenChart>
    <RadzenColumnSeries Data="@chartData" CategoryProperty="Category" ValueProperty="Value" />
</RadzenChart>
```

## Best Practices

1. **Performance**: Use virtualization for large datasets
2. **Accessibility**: Ensure proper ARIA labels and keyboard navigation
3. **Responsive Design**: Use Radzen's responsive grid system
4. **State Management**: Use appropriate state management patterns
5. **Error Handling**: Implement proper error handling and user feedback
6. **Testing**: Write unit tests for your Blazor components

## Common Issues and Solutions

1. **JavaScript Interop Issues**: Ensure Radzen.Blazor.js is loaded
2. **CSS Not Loading**: Check the CSS file path and ensure static files are configured
3. **Service Not Registered**: Verify all Radzen services are registered in Program.cs
4. **Form Validation**: Use Radzen validators with proper component names

## Verification Steps

After setup, verify the installation by:

1. Building the project successfully
2. Running the application
3. Testing basic Radzen components (buttons, forms, data grid)
4. Checking browser console for JavaScript errors
5. Verifying CSS styles are applied correctly