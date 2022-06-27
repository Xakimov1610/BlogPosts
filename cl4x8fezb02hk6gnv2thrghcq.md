## C# - Generic Repository Pattern

Entity Framework 6 da Generic Repository Patterndan foydalanish.

**Generic haqida tushuncha:**

.Net 2.0 dan boshlab Genericlardan foydalanish imkoniyati qo’shildi. Generic yordamida ixtiyoriy tipda ishlovchi(ixtiyoriy tipga moslashuvchi) metod yoki class yaratish mumkin. Bunda class yoki metod tipi uchun maxsus shablon belgisi — «T» dan foydalaniladi. Kompilyator har bir tipga mos shu tipdagi metod yoki classni generatsiya qiladi. Generic dan foydalanish uchun “using” bo’limiga using System.Collections.Generic; ni qo’shish lozim.

Masalan:
1)

```cs
using System;
using System.Collections.Generic;

namespace GenericApplication
{
    public class MyGenericArray<T>
    {
        private T[] array;
        public MyGenericArray(int size)
        {
            array = new T[size + 1];
        }
        public T getItem(int index)
        {
            return array[index];
        }
        public void setItem(int index, T value)
        {
            array[index] = value;
        }
    }
           
    class Tester
    {
        static void Main(string[] args)
        {
            //Butun tipdagi massivni e’lon qilish:
            MyGenericArray<int> intArray = new MyGenericArray<int>(5);
            //Massiv elementlariga qiymat berish:
            for (int c = 0; c < 5; c++)
            {
                intArray.setItem(c, c*5);
            }
            //Massiv elementlarini chop qilish
	    Console.Write("Butun tipli massiv elementlari:");
            for (int p = 0; p < 5; p++)
            {
                Console.Write(intArray.getItem(p) + " ");
            }
            Console.WriteLine();
            //Belgi tipidagi massivni hosil qilish:
            String myName=”Nosirbek”;
            MyGenericArray<char> charArray = new MyGenericArray<char>(5);
            for (int c = 0; c < myName.Length; c++)
            {
                charArray.setItem(c, myName[c]);
            }
            Console.Write("Belgi tipidagi massiv elementlari:");
            for (int p = 0; p< myName.Length; p++)
            {
                Console.Write(charArray.getItem(p) + " ");
            }
            Console.WriteLine();
            Console.ReadKey();
        }
    }
}
```

Dasturni ishga tushirgach quyidagi natijani olishimiz mumkin:

0 5 10 15 20
N o s i r b e k

Generic metodlar:
Generic yordamida universal tip parametriga ega bo’lgan metod yaratishimiz mumkin
Masalan:
2)

```cs
using System;
using System.Collections.Generic;
namespace GenericTest1
{
    class Program
    {
        private static void swap<K>(ref K p1,ref K p2)
        {
            Console.WriteLine("p1={0}, p2={1}",p1,p2);
            K temp = p1;
            p1 = p2;
            p2 = temp;
            Console.WriteLine("p1={0}, p2={1}", p1, p2);
            Console.WriteLine("----------------------");
        }
        static void Main(string[] args)
        {
            int a = 10, b = 25;
            swap(ref a, ref b);
            string firstName = "Mansur", lastName = "Kurtov";
            swap(ref firstName, ref lastName);
            Console.ReadLine();
        }
    }
}
```

Natija:


Generic Repository Pattern:
1) Yangi Consule Application yaratamiz va unda “Models” deb nomlangan papka hosil qilamiz. Uning ichiga Person.cs nomli classni qo’shamiz.

`Pesron.cs:`

```cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
namespace EntityCrudApp2.Model
{
    [Table("Pesons", Schema = "dbo")] //Bazadagi jadval nomi – “Persons”
    public class Person
    {

        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)] //Primery key
        public int Id { get; set; }

        [Required]
        [StringLength(250)] //'Not nul'(bo'sh qiymat qabul qilmaydi) va uzunligi 250 bo’lgan belgi
        public string Name { get; set; }
    }
}

```

2) Proyekt ichida “Repository” papkasini hosil qilamiz va unga “IMyRepository” interfeysini va “MyRepository” classini qo’shamiz.

`IMyRepository.cs:`


```cs
using System;
using System.Data.Entity;
using System.Linq;
using System.Linq.Expressions;

namespace EntityCrudApp2.Repository
{
    public interface IMyRepository<T> : IDisposable where T : class
    {

        DbContext Context { get; }

        IQueryable<T> All();

        IQueryable<T> Filter(Expression<Func<T, bool>> predicate);

        bool Contains(Expression<Func<T, bool>> predicate);

        T Find(params object[] keys);

        T Find(Expression<Func<T, bool>> expression);

        T Create(T t);

        int Delete(T t);

        int Delete(Expression<Func<T, bool>> predicate);

        int Update(T t);

        void Execute(string command, params object[] parameters);

        void SaveChanges();

    }
}
```


`MyRepository.cs:`


```cs
using System;
using System.Data.Entity;
using System.Linq;
using System.Linq.Expressions;
using EntityCrudApp2.DB;

namespace EntityCrudApp2.Repository
{
    public class MyRepository<T> : IMyRepository<T> where T : class
    {
        public DbContext Context { get; private set; }
    
        public MyRepository()
            : this(new DbContext())
        {
            String s;
            int k=s.Length
        }

        public MyRepository(DbContext context)
        {
            Context = context;
        }

        public IQueryable<T> All()
        {
            return Filter(null);
        }

        public IQueryable<T> Filter(Expression<Func<T, bool>> predicate)
        {
            var query = Context.Set<T>();

            return predicate!=null ? query.Where(predicate) : query;
        }

        public bool Contains(Expression<Func<T, bool>> predicate)
        {
            return Find(predicate) != null;
        }

        public T Find(params object[] keys)
        {
            return Context.Set<T>().Find(keys);
        }

        public T Find(Expression<Func<T, bool>> expression)
        {
            return Context.Set<T>().FirstOrDefault(expression);
        }

        public T Create(T t)
        {
            Context.Entry(t).State = EntityState.Added;
            Context.Set<T>().Add(t);
            SaveChanges();

            return t;
        }

        public int Delete(T t)
        {
            Context.Entry(t).State = EntityState.Deleted;
            Context.Set<T>().Remove(t);
            SaveChanges();

            return 0;
        }

        public int Delete(Expression<Func<T, bool>> predicate)
        {
            var t = Find(predicate);

            Context.Entry(t).State = EntityState.Deleted;
            Context.Set<T>().Remove(t);
            SaveChanges();

            return 0;
        }

        public int Update(T t)
        {
            Context.Entry(t).State=EntityState.Modified;
            Context.Set<T>().Attach(t);
            SaveChanges();

            return 0;
        }

        public void Execute(string command, params object[] parameters)
        {
            Context.Database.ExecuteSqlCommand(command, parameters);
           // Stored proceduralarni ishga tushirish uchun ishlatiladi
            SaveChanges();
        }

        public void SaveChanges()
        {
            Context.SaveChanges();
        } 

        public void Dispose()
        {
            Context.Dispose();
        }
    }
}
```

Modelga mos jadvallani hosil qilish uchun qo’shimcha Configuration.cs classini hozil qilamiz. Buning uchun proyektga “Migrations” nomli papka qo’shamiz.

`Configuration.cs:`


```cs
namespace EntityCrudApp2.Migrations
{
    using System.Data.Entity.Migrations;
    internal sealed class Configuration : DbMigrationsConfiguration<EntityCrudApp2.DB.DbContext>
    {
        public Configuration()
        {
            AutomaticMigrationsEnabled = true;
        }        
    }
}
```


_“app.config”_ faylida Dbga bog’lanish uchun kerak bo’ladigan connectionString ni aniqlab qo’yamiz:

`app.config:`

```cs

<connectionStrings>
    <add name="myConnectionString" 
         connectionString="Data Source=.;Initial Catalog=MansurDB1;User Id=sa;password=mansur;MultipleActiveResultSets=True;App=EntityFramework" providerName="System.Data.EntityClient" />
  </connectionStrings>


```

Proyektda DB deb nomlangan papka hosil qilamiz va uning ichiga DbConfig.cs va DBContext.cs classlarini qo’shamiz:

`DbConfig.cs:`


```cs
using System.Data.Entity;
namespace EntityCrudApp2.DB
{
    public static class DbConfig
    {
       

        public static void Configure()
        { 
            Database.SetInitializer(new MigrateDatabaseToLatestVersion<DbContext, Migrations.Configuration>());
        }

    }
}
```

`DBContext.cs:`

```cs
using System.Configuration;
using System.Data.Entity;
using System.Data.Entity.ModelConfiguration.Conventions;
using EntityCrudApp2.Model;

namespace EntityCrudApp2.DB
{
    public class DBContext : DbContext
    {
        private static readonly string ConnectionString;

        static DBContext()
        {
            ConnectionString = ConfigurationManager.ConnectionStrings["myConnectionString "].ConnectionString;
        }

        public DbContext()
            : base(ConnectionString)
        {

        }

        public DbSet<Person> Persons { get; set; }

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
        }

    }
}
```

Endi yaratilgan Repositorydan foydalanib bazada model bo’yicha jadvalni hosil qilish va unda CRUD(Create, Read, Update, Delete) amallarini bajarishimiz mumkin:

`Program.cs:`

```cs
using System;
using EntityCrudApp2.DB;
using EntityCrudApp2.Model;
using EntityCrudApp2.Repository;

namespace EntityCrudApp2
{
    class Program
    {
        static void Main(string[] args)
        {
            DbConfig.Configure();

            using (IMyRepository<Person> repository = new MyRepository<Person>())
            {
                repository.Create(new Person
                {
                    Name = "Programmer 1"
                });
                Console.WriteLine("Bazaga qo'shilgan 'Person'lar ro'yxati:");

                foreach (var person in repository.All())
                {
                    Console.WriteLine(person.Name);
                }

            }
            Console.ReadLine();
        }
    }
}
```

Natija:

Bazaga qo'shilgan 'Person'lar ro'yxati:
Programmer 1