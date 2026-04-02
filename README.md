using Microsoft.AspNetCore.Mvc;
using UniversityCMS.Models;
using UniversityCMS.Services;

namespace UniversityCMS.Controllers
{
    public class AdminController : Controller
    {
        private readonly AdminService _adminService;

        public AdminController(AdminService adminService)
        {
            _adminService = adminService;
        }

        // ---- Auth ----

        [HttpGet]
        public IActionResult Login() => View();

        [HttpPost]
        public async Task<IActionResult> Login(string email, string password)
        {
            var admin = await _adminService.LoginAsync(email, password);
            if (admin == null)
            {
                ViewBag.Error = "Invalid email or password.";
                return View();
            }
            HttpContext.Session.SetInt32("AdminId", admin.AdminId);
            HttpContext.Session.SetString("Role", "Admin");
            return RedirectToAction("Dashboard");
        }

        public IActionResult Logout()
        {
            HttpContext.Session.Clear();
            return RedirectToAction("Login");
        }

        // ---- Dashboard ----

        public async Task<IActionResult> Dashboard()
        {
            if (HttpContext.Session.GetInt32("AdminId") == null) return RedirectToAction("Login");
            ViewBag.Faculties = await _adminService.GetAllFacultyAsync();
            ViewBag.Registrars = await _adminService.GetAllRegistrarsAsync();
            return View();
        }

        // ---- Add Faculty ----

        [HttpGet]
        public IActionResult AddFaculty()
        {
            if (HttpContext.Session.GetInt32("AdminId") == null) return RedirectToAction("Login");
            return View();
        }

        [HttpPost]
        public async Task<IActionResult> AddFaculty(Faculty faculty)
        {
            if (HttpContext.Session.GetInt32("AdminId") == null) return RedirectToAction("Login");
            await _adminService.AddFacultyAsync(faculty);
            TempData["Message"] = "Faculty added successfully.";
            return RedirectToAction("Dashboard");
        }

        // ---- Add Registrar ----

        [HttpGet]
        public IActionResult AddRegistrar()
        {
            if (HttpContext.Session.GetInt32("AdminId") == null) return RedirectToAction("Login");
            return View();
        }

        [HttpPost]
        public async Task<IActionResult> AddRegistrar(Registrar registrar)
        {
            if (HttpContext.Session.GetInt32("AdminId") == null) return RedirectToAction("Login");
            await _adminService.AddRegistrarAsync(registrar);
            TempData["Message"] = "Registrar added successfully.";
            return RedirectToAction("Dashboard");
        }

        [HttpGet]
        public IActionResult RemoveFaculty()
        {
            if (HttpContext.Session.GetInt32("AdminId") == null)
                return RedirectToAction("Login");

            return View();
        }

        [HttpPost]
        public async Task<IActionResult> RemoveFaculty(string name, string department)
        {
            var result = await _adminService.SearchFacultyAsync(name, department);
            return View(result);
        }

        public async Task<IActionResult> DeleteFaculty(int id)
        {
            await _adminService.DeleteFacultyAsync(id);
            TempData["Message"] = "Faculty removed successfully.";
            return RedirectToAction("Dashboard");
        }

        [HttpGet]
        public IActionResult RemoveRegistrar()
        {
            if (HttpContext.Session.GetInt32("AdminId") == null)
                return RedirectToAction("Login");

            return View();
        }

        [HttpPost]
        public async Task<IActionResult> RemoveRegistrar(string name)
        {
            var result = await _adminService.SearchRegistrarAsync(name);
            return View(result);
        }

        public async Task<IActionResult> DeleteRegistrar(int id)
        {
            await _adminService.DeleteRegistrarAsync(id);
            TempData["Message"] = "Registrar removed successfully.";
            return RedirectToAction("Dashboard");
        }
    }
}
