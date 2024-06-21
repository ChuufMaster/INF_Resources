# Ass3

## Back_End

### Auth

```cs

using Assignment2_Backend.Models;
using Assignment3_Backend.ViewModels;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

namespace Assignment3_Backend.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class AuthenticationController : ControllerBase
    {
        private readonly UserManager<AppUser> _userManager;
        private readonly IUserClaimsPrincipalFactory<AppUser> _claimsPrincipalFactory;
        private readonly IRepository _repository;
        private readonly IConfiguration _configuration;

        public AuthenticationController(UserManager<AppUser> userManager, IUserClaimsPrincipalFactory<AppUser> claimsPrincipalFactory, IRepository repository, IConfiguration configuration)
        {
            _repository = repository;
            _userManager = userManager;
            _claimsPrincipalFactory = claimsPrincipalFactory;
            _configuration = configuration;
        }

        [HttpPost]
        [Route("Register")]
        public async Task<IActionResult> Register(UserViewModel uvm)
        {
            var user = await _userManager.FindByIdAsync(uvm.emailaddress);

            if (user == null)
            {
                user = new AppUser
                {
                    Id = Guid.NewGuid().ToString(),
                    UserName = uvm.emailaddress,
                    Email = uvm.emailaddress
                };

                var result = await _userManager.CreateAsync(user, uvm.password);

                if (result.Errors.Count() > 0) return StatusCode(StatusCodes.Status500InternalServerError, "Internal Server Error. Please contact support.");
            }
            else
            {
                return Forbid("Account already exists.");
            }

            return Ok();
        }

        [HttpPost]
        [Route("Login")]
        public async Task<ActionResult> Login(UserViewModel uvm)
        {
            var user = await _userManager.FindByNameAsync(uvm.emailaddress);

            if (user != null && await _userManager.CheckPasswordAsync(user, uvm.password))
            {
                try
                {
                    var principal = await _claimsPrincipalFactory.CreateAsync(user);
                    return GenerateJWTToken(user);
                }
                catch (Exception)
                {
                    return StatusCode(StatusCodes.Status500InternalServerError, "Internal Server Error. Please contact support.");
                }
            }
            else
            {
                return NotFound("Does not exist");
            }
        }

        [HttpGet]
        private ActionResult GenerateJWTToken(AppUser user)
        {
            // Create JWT Token
            var claims = new[]
            {
                new Claim(JwtRegisteredClaimNames.Sub, user.Email),
                new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
                new Claim(JwtRegisteredClaimNames.UniqueName, user.UserName)
            };

            var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Tokens:Key"]));
            var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

            var token = new JwtSecurityToken(
                _configuration["Tokens:Issuer"],
                _configuration["Tokens:Audience"],
                claims,
                signingCredentials: credentials,
                expires: DateTime.UtcNow.AddHours(3)
            );

            return Created("", new
            {
                token = new JwtSecurityTokenHandler().WriteToken(token),
                user = user.UserName
            });
        }

    }
}
```

### Report controlling

```cs

using Assignment3_Backend.Models;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using System.Dynamic;

namespace Assignment3_Backend.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ReportController : ControllerBase
    {

        private readonly IRepository _repository;
        public ReportController(IRepository repository)
        {
            _repository = repository;
        }

        [HttpGet]
        [Route("ProductsReport")]
        public async Task<ActionResult<dynamic>> ProductsReport()
        {
            try
            {
                List<dynamic> productsreport = new List<dynamic>();

                var results = await _repository.GetProductsReportAsync();

                dynamic brands = results
                             .GroupBy(p => p.Brand.Name)
                             .Select(b => new
                             {
                                 Key = b.Key,
                                 ProductCount = b.Count()
                             ,
                                 ProductTotalCost = Math.Round((double)b.Sum(p => p.Price), 2)
                             ,
                                 ProductAverageCost = Math.Round((double)b.Average(p => p.Price), 2)
                             });

                dynamic productTypes = results
                             .GroupBy(p => p.ProductType.Name)
                             .Select(pt => new
                             {
                                 Key = pt.Key,
                                 ProductCount = pt.Count()
                             ,
                                 ProductTotalCost = Math.Round((double)pt.Sum(p => p.Price), 2)
                             ,
                                 ProductAverageCost = Math.Round((double)pt.Average(p => p.Price), 2)
                             });

                dynamic productList = results
                    .GroupBy(p => new { BrandName = p.Brand.Name, ProductTypeName = p.ProductType.Name, ProductName = p.Name })
                    .Select(p => new
                    {
                        p.Key.BrandName,
                        p.Key.ProductTypeName,
                        p.Key.ProductName,
                        ProductPrice = Math.Round((double)p.Sum(x => x.Price), 2)
                    });

                productsreport.Add(brands);
                productsreport.Add(productTypes);
                productsreport.Add(productList);
               

                return productsreport;
            }
            catch (Exception)
            {
                return StatusCode(StatusCodes.Status500InternalServerError, "Internal Server Error. Please contact support.");
            }
        }
    }
}
```

### Store Controller

```cs

using Assignment3_Backend.Models;
using Assignment3_Backend.ViewModels;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Microsoft.IdentityModel.Tokens;
using System;
using System.Globalization;
using System.IdentityModel.Tokens.Jwt;
using System.IO;
using System.Linq;
using System.Net.Http.Headers;
using System.Runtime.InteropServices;
using System.Security.Claims;
using System.Text;
using System.Threading.Tasks;

namespace Assignment3_Backend.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class StoreController : ControllerBase
    {
        private readonly IRepository _repository;
        public StoreController(IRepository repository)
        {
            _repository = repository;
        }

        [HttpGet]
        [Route("ProductListing")]
        public async Task<ActionResult> ProductListing()
        {
            try
            {
                var results = await _repository.GetProductsAsync();

                dynamic products = results.Select(p => new
                {
                    p.ProductId,
                    p.Price,
                    ProductTypeName = p.ProductType.Name,
                    BrandName = p.Brand.Name,
                    p.Name,
                    p.Description,
                    p.DateCreated,
                    p.DateModified,
                    p.IsActive,
                    p.IsDeleted,
                    p.Image
                });

                return Ok(products);
            }
            catch (Exception)
            {

                return StatusCode(StatusCodes.Status500InternalServerError, "Internal Server Error. Please contact support.");
            }
        }

        [HttpPost, DisableRequestSizeLimit]
        [Route("AddProduct")]
        public async Task<IActionResult> AddProduct([FromForm] IFormCollection formData)
        {
            try
            {
                var formCollection = await Request.ReadFormAsync();
                
                var file = formCollection.Files.First();
                
                if (file.Length > 0)
                {
                    
                    using (var ms = new MemoryStream())
                    {
                        file.CopyTo(ms);
                        var fileBytes = ms.ToArray();
                        string base64 = Convert.ToBase64String(fileBytes);

                        string price = formData["price"];
                        decimal num = decimal.Parse(price.Replace(".", ","));

                        var product = new Product
                        {
                            Price = num
                            ,
                            Name = formData["name"]
                            ,
                            Description = formData["description"]
                            ,
                            BrandId = Convert.ToInt32(formData["brand"])
                            ,
                            ProductTypeId = Convert.ToInt32(formData["producttype"])
                            ,
                            Image = base64
                            ,
                            DateCreated = DateTime.Now
                        };


                        _repository.Add(product);
                        await  _repository.SaveChangesAsync();                        
                    }

                    return Ok();
                }
                else
                {
                    return BadRequest();
                }
            }
            catch (Exception ex)
            {
                return StatusCode(500, $"Internal server error: {ex}");
            }
        }


        [HttpGet]
        [Route("Brands")]
        public async Task<ActionResult> Brands()
        {
            try
            {
                var results = await _repository.GetBrandsAsync();

                return Ok(results);
            }
            catch (Exception)
            {

                return StatusCode(StatusCodes.Status500InternalServerError, "Internal Server Error. Please contact support.");
            }
        }


        [HttpGet]
        [Route("ProductTypes")]
        public async Task<ActionResult> ProductTypes()
        {
            try
            {
                var results = await _repository.GetProductTypesAsync();

                return Ok(results);
            }
            catch (Exception)
            {

                return StatusCode(StatusCodes.Status500InternalServerError, "Internal Server Error. Please contact support.");
            }
        }

    }
}
```

### Models

```cs

namespace Assignment3_Backend.Models
{
    public class Brand : BaseEntity
    {
        public int BrandId { get; set; }
        public virtual ICollection<Product> Products { get; set; }
    }
}
```

```cs

using Microsoft.EntityFrameworkCore.Metadata.Internal;
using System.ComponentModel.DataAnnotations.Schema;

namespace Assignment3_Backend.Models
{
    public class Product : BaseEntity
    {
        public int ProductId { get; set; }
        public decimal Price { get; set; }

        public string? Image { get; set; }
        public int BrandId { get; set; }
        public int ProductTypeId { get; set; }

        public ProductType ProductType { get; set; }
        public Brand Brand { get; set; }
    }
}
```

```cs

namespace Assignment3_Backend.Models
{
    public class ProductType : BaseEntity
    {
        public int ProductTypeId { get; set; }

        public virtual ICollection<Product> Products { get; set; }
    }
}
```

### Repository

```cs

using Microsoft.EntityFrameworkCore;

namespace Assignment3_Backend.Models
{
    public class Repository:IRepository
    {
        private readonly AppDbContext _appDbContext;

        public Repository(AppDbContext appDbContext)
        {
            _appDbContext = appDbContext;
        }

        public void Add<T>(T entity) where T : class
        {
            _appDbContext.Add(entity);
        }
        public async Task<bool> SaveChangesAsync()
        {
            return await _appDbContext.SaveChangesAsync() > 0;
        }

        public async Task<Product[]> GetProductsAsync()
        {
            IQueryable<Product> query = _appDbContext.Products.Include(p => p.Brand).Include(p => p.ProductType);

            return await query.ToArrayAsync();
        }

        public async Task<Brand[]> GetBrandsAsync()
        {
            IQueryable<Brand> query = _appDbContext.Brands.OrderBy(p => p.Name); ;

            return await query.ToArrayAsync();
        }

        public async Task<ProductType[]> GetProductTypesAsync()
        {
            IQueryable<ProductType> query = _appDbContext.ProductTypes;

            return await query.ToArrayAsync();
        }

        public async Task<Product[]> GetProductsReportAsync()
        {
            IQueryable<Product> query = _appDbContext.Products.Include(p => p.Brand).Include(p => p.ProductType).Where(p=> p.IsActive == true).OrderBy(p => p.Brand.Name);

            return await query.ToArrayAsync();
        }
    }
}
```

### ProgramCS

```cs

using Assignment3_Backend.Factory;
using Assignment3_Backend.Models;
using Microsoft.AspNetCore.Http.Features;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using Microsoft.OpenApi.Models;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.


builder.Services.AddCors(options => options.AddDefaultPolicy(
    include =>
    {
        include.AllowAnyHeader();
        include.AllowAnyMethod();
        include.AllowAnyOrigin();
    }));

builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddIdentity<AppUser, IdentityRole>(options =>
{
    options.Password.RequireUppercase = false;
    options.Password.RequireLowercase = false;
    options.Password.RequireNonAlphanumeric = false;
    options.Password.RequireDigit = true;
    options.User.RequireUniqueEmail = true;
})
.AddEntityFrameworkStores<AppDbContext>()
.AddDefaultTokenProviders();

builder.Services.AddAuthentication()
                .AddCookie()
                .AddJwtBearer(options =>
                {
                    options.TokenValidationParameters = new TokenValidationParameters()
                    {
                        ValidIssuer = builder.Configuration["Tokens:Issuer"],
                        ValidAudience = builder.Configuration["Tokens:Audience"],
                        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Tokens:Key"]))
                    };
                });

builder.Services.Configure<FormOptions>(o =>
{
    o.ValueLengthLimit = int.MaxValue;
    o.MultipartBodyLengthLimit = int.MaxValue;
    o.MemoryBufferThreshold = int.MaxValue;
});

builder.Services.AddScoped<IUserClaimsPrincipalFactory<AppUser>, AppUserClaimsPrincipalFactory>();

builder.Services.Configure<DataProtectionTokenProviderOptions>(options => options.TokenLifespan = TimeSpan.FromHours(3));

builder.Services.AddDbContext<AppDbContext>(options => options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
builder.Services.AddScoped<IRepository, Repository>();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseCors();
app.UseAuthorization();
app.UseAuthentication();



app.MapControllers();

app.Run();

```
