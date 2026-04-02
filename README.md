using Microsoft.EntityFrameworkCore;
using UniversityCMS.Data;
using UniversityCMS.Models;

namespace UniversityCMS.Services
{
    public class AdminService
    {
        private readonly ApplicationDbContext _db;

        public AdminService(ApplicationDbContext db)
        {
            _db = db;
        }

        public async Task<Admin?> LoginAsync(string email, string password)
            => await _db.Admins.FirstOrDefaultAsync(a => a.Email == email && a.Password == password);

        public async Task AddFacultyAsync(Faculty faculty)
        {
            _db.Faculties.Add(faculty);
            await _db.SaveChangesAsync();
        }

        public async Task AddRegistrarAsync(Registrar registrar)
        {
            _db.Registrars.Add(registrar);
            await _db.SaveChangesAsync();
        }

        public async Task<List<Faculty>> GetAllFacultyAsync()
            => await _db.Faculties.ToListAsync();

        public async Task<List<Registrar>> GetAllRegistrarsAsync()
            => await _db.Registrars.ToListAsync();

        // ---- Search Faculty ----
        public async Task<List<Faculty>> SearchFacultyAsync(string name, string department)
        {
            var query = _db.Faculties.AsQueryable();

            if (!string.IsNullOrEmpty(name))
                query = query.Where(f => f.Name.Contains(name));

            if (!string.IsNullOrEmpty(department))
                query = query.Where(f => f.Department.Contains(department));

            return await query.ToListAsync();
        }

        // ---- Delete Faculty ----
        public async Task DeleteFacultyAsync(int id)
        {
            var faculty = await _db.Faculties.FindAsync(id);
            if (faculty != null)
            {
                _db.Faculties.Remove(faculty);
                await _db.SaveChangesAsync();
            }
        }


        // ---- Search Registrar ----
        public async Task<List<Registrar>> SearchRegistrarAsync(string name)
        {
            var query = _db.Registrars.AsQueryable();

            if (!string.IsNullOrEmpty(name))
                query = query.Where(r => r.Name.Contains(name));

            return await query.ToListAsync();
        }


        // ---- Delete Registrar ----
        public async Task DeleteRegistrarAsync(int id)
        {
            var registrar = await _db.Registrars.FindAsync(id);
            if (registrar != null)
            {
                _db.Registrars.Remove(registrar);
                await _db.SaveChangesAsync();
            }
        }
    }
}
