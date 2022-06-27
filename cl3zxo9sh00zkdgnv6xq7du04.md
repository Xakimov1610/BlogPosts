## .Net 6 da xUnit va MOQ yordamida Unit test

Ushbu maqolada biz Unit Testi va uni .Net 6 da qanday amalga oshirish mumkinligini bilib olamiz.

_Shuningdek, siz GitHub-da mavjud bo'lgan source code ni topishingiz mumkin._
[github suorce code](https://github.com/Xakimov1610/Mohamad-Lawand)

## Unit test nima?
Unit Test tizimda logik ravishda ajratilishi mumkin bo'lgan eng kichik kod qismidir, odatda biz logik izolyatsiya qilingan kodning eng kichik qismini o'z funcsialarimiz deb hisoblaymiz. Ushbu kichik kod parchasi bilan biz avtomatlashtirilgan testlarni o'tkazishimiz mumkin, bu bizning kodimiz har doim to'g'ri natija berishiga ishonch hosil qiladi.
## Nima uchun kodimni Test qilishim kerak?
- Vaqtni tejaydi: ba'zi hollarda qo'lda test o'tkazish zaruratini yo'qotadi
- Avtomatlashtirish: o'zgartirilgan kodni tezda qayta sinab ko'rish imkoniyati
- **Samarali kod: Barcha potentsial stsenariylar qamrab olinganligiga ishonch hosil qiling va Unit Testni  amalga oshirish uchun biz kodimizni haqiqatda sinab ko'rishimiz uchun tuzilishi kerak. Bu shuni anglatadiki, biz SOLID kabi ma'lum tamoyillarga amal qilishimiz kerak.**
- Documentation: Bizning methodlarimiz ortida turgan logikani tushunishga yordam beradi.
- **Sifat: Code bazamiz sifatini yaxshilash, texnik qarzni iloji boricha ko'paytirmaslikka yordam beradi**
- Ishonchli: Sinovlar o'tayotganda va avtomatik ravishda bajarilganda, biz yanada yumshoqroq va tez-tez **relizlarga** ega bo'lamiz.
## Qaysi Unit testdan foydalanish kerak?
- msTest - etuk, sekin
- NUnit - etuk, xususiyatli, tezkor
- xUnit.net - yangi, tez

**Console application yaratish.**

```csharp
dotnet new console -n "TestingBasic"
```
**Biz ba'zi nuget paketlarini o'rnatishimiz kerak**

```csharp
dotnet add package Microsoft.NET.Test.Sdk 
dotnet add package xunit 
dotnet add package xunit.runner.visualstudio 
dotnet add package coverlet.collector
```

Functionalities.cs file yaratib, UserManagement deb nomlangan Class qo’shing va quyidagilarni qo'shing

```csharp
namespace TestingBasics.Functionalities;

public record User(string FirstName, string LastName)
{
    public int Id { get; init; }
    public DateTime CreatedDate { get; init; } = DateTime.UtcNow;
    public string Phone { get; set; } = "+44 ";
    public bool VerifiedEmail { get; set; } = false;
}

public class UserManagement
{
    private readonly List<User> _users = new ();
    private int idCounter = 1;

    public IEnumerable<User> AllUsers => _users;

    public void Add(User user)
    {
        _users.Add(user with {Id = idCounter++});
    }

    public void UpdatePhone(User user)
    {
        var dbUser = _users.First(x => x.Id == user.Id);

        dbUser.Phone = user.Phone;
    }

    public void VerifyEmail(int id)
    {
        var dbUser = _users.First(x => x.Id == id);

        dbUser.VerifiedEmail = true;
    }
```

Testlar va ilovani bitta Projectda o'tkazishimiz mumkin bo'lsa-da, testni alohida ilovada o'tkazish tavsiya etiladi, bu esa muammolarni yaxshiroq ajratishga olib keladi.

Biz yangi xunit loyihamizni quyidagi buyruq bilan yaratamiz

```csharp
dotnet new xunit -n "TestingBasics.Test"
```

Loyiha muvaffaqiyatli yaratilgandan so'ng, biz ilovamizga reference qo'shishimiz kerak

```csharp
dotnet add TestingBasics.Test/TestingBasics.Test.csproj reference TestingBasics/TestingBasics.csproj
```

Endi UserManagementTest deb nomlangan yangi Class yarating va biz quyidagilarni qo'shamiz

```csharp
using System.Linq;
using TestingBasics.Functionalities;
using Xunit;

namespace TestingBasics.Test;

public class UserManagementTest
{
    [Fact]
    public void Add_CreateUser()
    {
        // Arrange
        var userManagement = new UserManagement();

        // Act
        userManagement.Add(new(
                "Nosirbek", "Xakimov"
        ));

        // Assert
        var savedUser = Assert.Single(userManagement.AllUsers);
        Assert.NotNull(savedUser);
        Assert.Equal("Nosirbek", savedUser.FirstName);
        Assert.Equal("Xakimov", savedUser.LastName);
        Assert.NotEmpty(savedUser.Phone);
        Assert.False(savedUser.VerifiedEmail);
    }

    [Fact]
    public void Verify_VerifyEmailAddress()
    {
        // Arrange
        var userManagement = new UserManagement();

        // Act
        userManagement.Add(new(
                "Nosirbek", "Xakimov"
        ));

        var firstUser = userManagement.AllUsers.ToList().First();
        userManagement.VerifyEmail(firstUser.Id);

        // Assert
        var savedUser = Assert.Single(userManagement.AllUsers);
        Assert.True(savedUser.VerifiedEmail);
    }

    [Fact]
    public void Update_UpdateMobileNumber()
    {
        // Arrange
        var userManagement = new UserManagement();

        // Act
        userManagement.Add(new(
                "Nosirbek", "Xakimov"
        ));

        var firstUser = userManagement.AllUsers.ToList().First();
        firstUser.Phone = "+998901203949";
        userManagement.UpdatePhone(firstUser);

        // Assert
        var savedUser = Assert.Single(userManagement.AllUsers);
        Assert.Equal("+998901203949",savedUser.Phone);
    }
}
```

Sizda Microsoft.NET.Test.Sdk versiyasining 17.0.0 so‘nggi versiyasi mavjudligiga ishonch hosil qiling TestingBasics.Test.csproj-ni oching va versiyani tekshiring, agar 17.0.0 dan pastroq bo‘lsa, uni eng so‘nggi versiyaga yangilang.

Endi testimizni o'tkazish vaqti keldi

```csharp
dotnet test
```

### **Mock**

Keling,keyingi bosqichga o'tamiz va Mock-dan qanday foydalanishimiz mumkinligini tekshiramiz, shuning uchun Mocking nima? Agar service boshqa servicega depends bo'lsa va biz ushbu serviceni Test qilib ko'rmoqchi bo'lsak ‘Mock’ yan Soxta dan foydalanamiz.

Ikkinchi service to'liq ishga tushirish jarayonini amalga oshirish o'rniga, biz uni Mock qilishimiz mumkin (go'yo u to'liq ishlagandek ko'rsatamiz) va shunga asoslanib testlarimizni o'tkazishimiz mumkin.

Keling, ShoppingCart deb nomlangan yangi Class yaratamiz va quyidagilarni qo'shamiz

```csharp
namespace TestingBasics.Functionalities;

// Mahsulotni aniqlash
public record Product(int Id, string Name, double price);

// Db Service
public interface IDbService
{
    bool SaveShoppingCartItem(Product prod);
    bool RemoveShoppingCartItem(int? id);
}

// Shopping Cart functionality
public class ShoppingCart
{
    private IDbService _dbService;

    public ShoppingCart(IDbService dbService)
    {
        _dbService = dbService;
    }

    public bool AddProduct(Product? product)
    {
        if(product == null)
            return false;

        if(product.Id == 0)
            return false;

        _dbService.SaveShoppingCartItem(product);
        return true;    
    }

    public bool DeleteProduct(int? id)
    {
        if(id == null)
            return false;

        if(id == 0)
            return false;

        _dbService.RemoveShoppingCartItem(id);
        return true;    
    }
}
```

Keling, ushbu functionalities uchun Unit testini yaratishni boshlaylik. Ko'rib turganingizdek, bu functionality Db xizmatiga bog'liq, ya'ni biz yozgan ilovalarni sinab ko'rishimiz uchun ushbu xizmatni Mock qilishimiz kerak.

Test Class ichida ShoppingCartTest nomli yangi Class yaratamiz. Va biz Mock tushunchasini tushunganimizdan so'ng, kutubxonadan foydalanish uchun uni mavhumlashtirishimiz mumkin bo'lgan narsalar qanday ishlashini batafsil ko'rish uchun har qanday kutubxonadan foydalanish o'rniga qo'lda Mockni amalga oshirishni boshlaymiz.

```csharp
using System.Linq;
using TestingBasics.Functionalities;
using Xunit;
using System;

namespace TestingBasics.Test;

public class ShoppingCartTest
{
    public class DbServiceMock : IDbService
    {
        public bool ProcessResult { get; set; }
        public Product ProductBeingProcessed {get;set;}
        public int ProductIdBeingProcessed { get; set; }

        public bool RemoveShoppingCartItem(int? id)
        {
            if(id != null)
            ProductIdBeingProcessed = Convert.ToInt32(id);
            return ProcessResult;
        }

        public bool SaveShoppingCartItem(Product prod)
        {
            ProductBeingProcessed = prod;
            return ProcessResult;
        }
    }

    [Fact]
    public void AddProduct_Success()
    {
        var dbMock = new DbServiceMock();
        dbMock.ProcessResult = true;
        // Arrange  | Tartibga solish
        ShoppingCart shoppingCart = new (dbMock);

        // Act | Qoida
        var product = new Product(1, "Shoes", 200);
        var result = shoppingCart.AddProduct(product);

        // Assert | Tasdiqlash
        Assert.True(result);
        Assert.Equal(product, dbMock.ProductBeingProcessed);
    }

    [Fact]
    public void AddProduct_Failure_InvalidPayload()
    {
         var dbMock = new DbServiceMock();
        dbMock.ProcessResult = false;

        // Arrange
        ShoppingCart shoppingCart = new (dbMock);

        // Act
        var result = shoppingCart.AddProduct(null);

        // Assert
        Assert.False(result);
    }

    [Fact]
    public void RemoveProduct_Success()
    {
        var dbMock = new DbServiceMock();
        dbMock.ProcessResult = true;
        // Arrange
        ShoppingCart shoppingCart = new (dbMock);

        // Act
        var product = new Product(1, "Shoes", 200);
        var result = shoppingCart.DeleteProduct(product.Id);

        // Assert
        Assert.True(result);
        Assert.Equal(product.Id, dbMock.ProductIdBeingProcessed);
    }

    [Fact]
    public void RemoveProduct_Failed()
    {
        var dbMock = new DbServiceMock();
        dbMock.ProcessResult = false;
        // Arrange
        ShoppingCart shoppingCart = new (dbMock);

        // Act
        var result = shoppingCart.DeleteProduct(null);

        // Assert
        Assert.False(result);
    }

    [Fact]
    public void RemoveProduct_Failed_InvalidId()
    {
        var dbMock = new DbServiceMock();
        dbMock.ProcessResult = false;
        // Arrange
        ShoppingCart shoppingCart = new (dbMock);

        // Act
        var result = shoppingCart.DeleteProduct(0);

        // Assert
        Assert.False(result);
    }
}
```

Xizmatlarimizdan birini qo'lda Mock qilish uchun qancha qo'l mehnati kerakligini tatib ko'rganimizdan so'ng, mocking kutubxona bizga qanday yordam berishini tekshirish vaqti keldi.

Biz Moq nuget paketidan foydalanmoqchimiz, shuning uchun quyidagi nugetni o'rnatish orqali uni loyihamizga qo'shamiz.

```csharp
dotnet add package Moq
```

Endi paketni o'rnatganimizdan so'ng, uni kodimizga kiritish vaqti keldi, keling, Unit test kodini quyidagiga yangilaylik.

```csharp
using System.Linq;
using TestingBasics.Functionalities;
using Xunit;
using System;
using Moq;

namespace TestingBasics.Test;

public class ShoppingCartTest
{
    public readonly Mock<IDbService> _dbServiceMock = new();

    [Fact]
    public void AddProduct_Success()
    {
        var product = new Product(1, "Shoes", 200);
        _dbServiceMock.Setup(x => x.SaveShoppingCartItem(product)).Returns(true);
        // Arrange
        ShoppingCart shoppingCart = new (_dbServiceMock.Object);

        // Act
        var result = shoppingCart.AddProduct(product);

        // Assert
        Assert.True(result);
        _dbServiceMock.Verify(x => x.SaveShoppingCartItem(It.IsAny<Product>()), Times.Once);
    }

    [Fact]
    public void AddProduct_Failure_InvalidPayload()
    {

        // Arrange
        ShoppingCart shoppingCart = new (_dbServiceMock.Object);

        // Act
        var result = shoppingCart.AddProduct(null);

        // Assert
        Assert.False(result);
        _dbServiceMock.Verify(x => x.SaveShoppingCartItem(It.IsAny<Product>()), Times.Never);
    }

    [Fact]
    public void RemoveProduct_Success()
    {
        var product = new Product(1, "Shoes", 200);
        _dbServiceMock.Setup(x => x.RemoveShoppingCartItem(product.Id)).Returns(true);

        // Arrange
        ShoppingCart shoppingCart = new (_dbServiceMock.Object);

        // Act
        var result = shoppingCart.DeleteProduct(product.Id);

        // Assert
        Assert.True(result);
        _dbServiceMock.Verify(x => x.RemoveShoppingCartItem(It.IsAny<int>()), Times.Once);
    }

    [Fact]
    public void RemoveProduct_Failed()
    {
        _dbServiceMock.Setup(x => x.RemoveShoppingCartItem(null)).Returns(false);

        // Arrange
        ShoppingCart shoppingCart = new (_dbServiceMock.Object);

        // Act
        var result = shoppingCart.DeleteProduct(null);

        // Assert
        Assert.False(result);
        _dbServiceMock.Verify(x => x.RemoveShoppingCartItem(null), Times.Never);
    }

    [Fact]
    public void RemoveProduct_Failed_InvalidId()
    {
        _dbServiceMock.Setup(x => x.RemoveShoppingCartItem(null)).Returns(false);

        // Arrange
        ShoppingCart shoppingCart = new (_dbServiceMock.Object);

        // Act
        var result = shoppingCart.DeleteProduct(0);

        // Assert
        Assert.False(result);
        _dbServiceMock.Verify(x => x.RemoveShoppingCartItem(null), Times.Never);
    }
}
```

```csharp
dotnet test
```


![Image description](https://cdn.hashnode.com/res/hashnode/image/upload/v1654350664163/ci-xZkhlt.png)



