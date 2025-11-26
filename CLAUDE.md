# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Diayma** is an ASP.NET Core MVC e-commerce application built on .NET Core 2.0. This is a learning/training project originally targeting an obsolete framework version (.NET Core 2.0, EOL 2018).

**Solution Structure:**
- Single project: `Diayma` (located in `P2FixAnAppDotNetCode/`)
- Solution file: `Diayma.sln`

## Build and Run Commands

### Development
```bash
# Build the project
dotnet build

# Run the application
dotnet run --project P2FixAnAppDotNetCode/Diayma.csproj

# Watch mode for development (auto-reload on changes)
dotnet watch run --project P2FixAnAppDotNetCode/Diayma.csproj
```

### Self-Contained Publishing (Windows)
The app can be published as a standalone executable that includes the .NET runtime:
```bash
dotnet publish -c Release -r win-x64 --self-contained
```
Output: `P2FixAnAppDotNetCode/bin/Release/netcoreapp2.0/win-x64/publish/Diayma.exe`

## Architecture

### Application Flow
1. **Entry Point**: `Program.cs:Main()` → `BuildWebHost()` creates `IWebHost`
2. **Startup**: `Startup.cs`
   - `ConfigureServices()`: DI configuration, MVC, localization, session
   - `Configure()`: Request pipeline with static files → localization → session → MVC
3. **Default Route**: `/{controller=Product}/{action=Index}/{id?}`
4. **View Rendering**: Razor views with `_Layout.cshtml` and ViewComponents

### Dependency Injection (Startup.cs:26-34)
- **Singleton**: `ICart` → `Cart`, `ILanguageService` → `LanguageService`
- **Transient**: Product/Order services and repositories
- **Infrastructure**: Session (in-memory), Localization, MVC with view/data annotation localization

### Domain Model

**Core Entities:**
- `Product`: ID, Name, Description, Stock, Price (in-memory hardcoded list)
- `Order`: OrderId, Lines (CartLine[]), customer details (Name, Address, City, Zip, Country, Date)
- `Cart` (ICart): Shopping cart with `CartLine` items
- `CartLine`: OrderLineId, Product, Quantity

**Repositories (in-memory):**
- `ProductRepository`: Manages static product list, filters by stock > 0
- `OrderRepository`: Stores orders in static list

**Services (business logic):**
- `ProductService`: Product retrieval, stock updates after checkout
- `OrderService`: Order creation, delegates to ProductService for inventory updates
- `LanguageService`: Culture switching for localization

### Controllers

**ProductController** (`ProductController.cs:15`)
- `Index()`: Lists all available products (stock > 0)

**CartController** (`CartController.cs:15`)
- `Index()`: Display cart
- `AddToCart(id)`: Add product to cart
- `RemoveFromCart(id)`: Remove product from cart

**OrderController** (`OrderController.cs:17`)
- `Index()` GET: Show checkout form
- `Index(Order)` POST: Validate and save order, update inventory
- `Completed()`: Order confirmation, clears cart

**LanguageController**: Switches UI culture (en/fr)

### View Components

**CartSummaryViewComponent** (`CartSummaryViewComponent.cs:12`)
- Displays cart summary in layout header
- Shows item count

**LanguageSelectorViewComponent**
- Language switcher dropdown in layout

### Localization

**Supported Cultures** (Startup.cs:45-52):
- English: `en`, `en-US`, `en-GB`
- French: `fr`, `fr-FR`

**Resource Files** (`Resources/` directory):
- View-specific: `Views/{Controller}/{View}.{culture}.resx`
- Model validation: `Models/ViewModels/{Model}.{culture}.resx`
- Controller messages: `Controllers/{Controller}.{culture}.resx`

**Custom Wolof Support** (README.md:60-62):
- Client-side JSON file: `wwwroot/i18n/wo.json`
- Selector in `_Layout.cshtml`

## Known Issues

### Critical: Obsolete Framework
- **Target**: .NET Core 2.0 (EOL since October 2018)
- **Impact**: Cannot run without .NET Core 2.0 runtime
- **Workaround**: Self-contained deployment bundles runtime

### Security: Vulnerable Packages
- Multiple NuGet packages have known vulnerabilities
- Microsoft.AspNetCore.All 2.0.6 includes outdated dependencies
- Newtonsoft.Json < 13.0.2 has security issues

## Development Notes

### Debugging Breakpoints (from README.md:23-28)
Common breakpoints for request flow:
- `Startup.cs:20` - Service configuration
- `ProductController.cs:15` - Product listing
- `CartController.cs:15` - Cart operations
- `OrderController.cs:17` - Checkout flow
- `CartSummaryViewComponent.cs:12` - Cart display

### Cart Implementation Details
- **Total Calculation Bug** (Cart.cs:66): Only sums product prices, ignores quantities
- **Average Calculation** (Cart.cs:77): Averages product prices, not cart value
- Both methods should multiply `Product.Price × CartLine.Quantity`

### In-Memory State
- Products: Static list in `ProductRepository` (ProductRepository.cs:25-34)
- Orders: Static list in `OrderRepository`
- Cart: Singleton service (shared across requests - not production-ready)

Session is configured but cart doesn't use it - cart state is application-wide singleton.
