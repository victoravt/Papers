# Intro

I've been working with WebApi for 5 years as well as with desktop applications and heavy SQL oriented applications mostly on enterprise . 
Non SQL - Elastic search, Azure Cognitive search
More details you can see in my profile. Let me not repeat myself.


## 1. *WebApi experience*: **5 years** of experience

> Simple WebApi Controller class as an example

```c#
// this is a real life application example that I maintain on leisure time
namespace ProjName.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ExerciseController : BaseController
    {
        private IExerciseService _exerciseService;
        private IFileService _fileService;

        public ExerciseController(IExerciseService exerciseService,
            IFileService fileService,
            IMapper mapper)
            : base(mapper)
        {
            _exerciseService = exerciseService;
            _fileService = fileService;
        }

        [HttpGet("get")]
        public async Task<ActionResult<ExerciseDto_v2>> GetById(long id)
        {
            var result = await _exerciseService.GetById(id);

            return result.ToActionResult();
        }

        [HttpGet("get-all")]
        public async Task<ActionResult<TableResult<ExerciseDto_ListView>>> GetAll(string? searchText, 
            int pageNumber = 0,
            [Range(0, 1000)]
            int pageSize = 10)
        {
            var all = await _exerciseService.GetAll(searchText, pageNumber, pageSize, LangOrDefault());
            return all.ToActionResult();
        }


        [HttpPost("create")]
        public async Task<ActionResult<ExerciseDto_v2>> Create(ExerciseRequestDto exercise)
        {
            var result = await _exerciseService.Create_v2(exercise);
            return result.ToActionResult();
        }

        [HttpPut("update")]
        public async Task<ActionResult<ExerciseDto_v2>> Update(ExerciseRequestDto exercise)
        {
            var result = await _exerciseService.Update_v2(exercise);
            return result.ToActionResult();
        }

        [HttpDelete("delete")]
        public async Task<ActionResult<ExerciseDto_v2>> Delete(long id)
        {
            var result = await _exerciseService.Delete(id);
            return result.ToActionResult();
        }

        [HttpDelete("delete-image")]
        public async Task<ActionResult<FileDto>> DeleteImage(long id)
        {
            var result = await _fileService.DeleteByIdFromDbAndPhysically(id);

            return result.ToActionResult();
        }

        /// <summary>
        /// Return "en" if no header "lang" is found or were empty.
        /// </summary>
        /// <returns></returns>
        private string LangOrDefault()
        {
            return !string.IsNullOrEmpty(Request.Headers["lang"]) ? Request.Headers["lang"].ToString() : "ru";
        }
    }
}
```

## 2. *Entity Framework*: **3 years** of experience
> Sample Data Access class using Entity Frame work as an example

```c#
// this is a real life application example that I maintain on leisure time
public class GenericRepository<T> : IGenericAsyncRepository<T> where T : BaseDomainObj
    {
        protected readonly AppDbContext _context;
        protected readonly DbSet<T> _table;
        public GenericRepository(AppDbContext context)
        {
            _context = context;
            _table = _context.Set<T>();
        }

        public IQueryable<T> GetQueryable(
            Expression<Func<T, bool>> predicate = null,
            Func<IQueryable<T>, IOrderedQueryable<T>> orderBy = null,
            Func<IQueryable<T>, IIncludableQueryable<T, object>> include = null,
            bool disableTracking = true)
        {
            //stugel
            IQueryable<T> query = _table;
            if (disableTracking)
            {
                query = query.AsNoTracking();
            }

            if (include != null)
            {
                query = include(query);
            }

            if (predicate != null)
            {
                query = query.Where(predicate);
            }

            IQueryable<T> result = orderBy == null ? query : orderBy(query);
            return result;
        }

        public async Task<List<T>> GetAllAsync(
            Expression<Func<T, bool>> predicate = null, 
            Func<IQueryable<T>,IOrderedQueryable<T>> orderBy = null, 
            Func<IQueryable<T>, IIncludableQueryable<T, object>> include = null,
            bool disableTracking = true
            )
        {
            //stugel
            IQueryable<T> query = _table;
            if(disableTracking)
            {
                query = query.AsNoTracking();
            }

            if(include != null)
            {
                query = include(query);
            }

            if(predicate != null)
            {
                query = query.Where(predicate);
            }

            List<T> result = ((orderBy == null) ? (await query.ToListAsync()) : (await orderBy(query).ToListAsync()));
            return result; 
        }

        public async Task<T?> GetFirstOrDefaultAsync(Expression<Func<T, bool>> predicate = null,
            Func<IQueryable<T>, IOrderedQueryable<T>> orderBy = null,
            Func<IQueryable<T>, IIncludableQueryable<T, object>> include = null,
            bool disableTracking = true
            )
        {
            IQueryable<T> query = _table;
            if (disableTracking)
            {
                query = query.AsNoTracking();
            }

            if (include != null)
            {
                query = include(query);
            }

            if (predicate != null)
            {
                query = query.Where(predicate);
            }

            if(orderBy != null)
            {
                return await orderBy(query).FirstOrDefaultAsync();
            }
            
            return await query.FirstOrDefaultAsync();
        }

        public async virtual Task<EntityEntry> AddAsync(T entity)
        {
            if (entity == null)
                throw new ArgumentNullException($"{nameof(this.AddAsync)} Paramenter {nameof(entity)} can't be null.");

            var entry = _table.Add(entity);
            var result = await _context.SaveChangesAsync();
            return entry;
        }

        public virtual async Task<EntityEntry> UpdateAsync(T entity)
        {
            if (entity == null)
                throw new ArgumentNullException($"{nameof(this.UpdateAsync)} Paramenter {nameof(entity)} can't be null.");

            var entry = _table.Update(entity);
            await _context.SaveChangesAsync();
            return entry;
        }

        public async Task<EntityEntry> DeleteAsync(T entity)
        {
            if (entity == null)
                throw new ArgumentNullException($"{nameof(this.DeleteAsync)} Paramenter {nameof(entity)} can't be null.");

            var entry = _table.Remove(entity);
            await _context.SaveChangesAsync();
            return entry;
        }

        public async virtual Task<EntityEntry> DeleteAsync(long id)
        {
            if (id <= 0)
                throw new ArgumentException($"{nameof(this.DeleteAsync)} Paramenter {nameof(id)} can't be less then or equal 0.");

            var entity = await GetFirstOrDefaultAsync(x => x.Id == id);
            if (entity == null)
                throw new ArgumentNullException($"No such entity in db with id {id}");

            var entry = _context.Remove(entity);
            await _context.SaveChangesAsync();
            return entry;
        }

        public async Task BulkDelete(IEnumerable<T> list)
        {
            if (list == null && !list.Any())
                throw new ArgumentException($"{nameof(this.BulkDelete)} Paramenter {nameof(list)} can't be empty.");

            _context.Set<T>().RemoveRange(list);
            await _context.SaveChangesAsync();
        }
    }
```

## 3. *Object Oriented Development*: **5 years** of experience

> OOP and Example Dependency injection

```c#
public class TrainingProgramService : BaseService, ITrainingProgramService
    {   
        private ITrainingProgramRepository _repo;
        private IMapper _mapper;
        private ITrainingProgramUnitRepository _unitRepo;
        private IResourceRepository _resourcesRepository;


        public TrainingProgramService(ITrainingProgramRepository repo,
            IResourceRepository resourcesRepository,
            ITrainingProgramUnitRepository unitRepo,
            IMapper mapper)
        {
            _repo = repo;
            _unitRepo = unitRepo;
            _mapper = mapper;
            _resourcesRepository = resourcesRepository;
        }

        public async Task<Result<TrainingProgramDto>> GetById(long id)
        {
            if(id <= 0)
                return Failure("Id should be greater than 0.");

            var entry = await _repo.GetFirstOrDefaultAsync(x => x.Id == id, include: TrainingProgramIncludes.Full);
            if (entry == default(TrainingProgramEntity))
                return NotFound<TrainingProgramDto>();

            return Success(_mapper.Map<TrainingProgramDto>(entry));
        }

        public async Task<Result<TableResult<TrainingProgramListViewDto>>> GetAllPaged(string? searchQuery, int? pageNumber, int? take,
            TrainingProgramsSort sort = TrainingProgramsSort.Default)
        {
            var query = _repo.GetQueryable(orderBy: TrainingProgramsSorting.Orders[sort], include: TrainingProgramIncludes.Full);
            SearchQuery(searchQuery, ref query);

            var totalCount = await query.CountAsync();

            int skip = 0;
            if (pageNumber != null && take != null)
            {
                skip = pageNumber.Value * take.Value;

                query = query.Skip(skip).Take(take.Value);
            }

            var trProgs = await query.ToListAsync();
            var exerciseIds = trProgs.SelectMany(s => s.UnitsOfProgram).Select(ss => ss.Exercise).Select(s => s.Id.ToString());

            var contents = await _resourcesRepository.Get(x => exerciseIds.Contains(x.PrimaryKey) && x.Lang == "en" && x.Type == ResourcesConst.Exercise.Type).ToListAsync();

            var grouppedResources = contents.GroupBy(x => x.PrimaryKey).ToDictionary(d => d.Key, d => d.ToList());

            var res = new List<TrainingProgramListViewDto>();

            var intermediateResult = _mapper.Map<List<TrainingProgramListViewDto>>(trProgs);

            // map resources into exercise results
            foreach (var ex in intermediateResult)
            {
                if (!grouppedResources.ContainsKey(ex.Id.ToString()))
                    continue;

                var correspondingResource = grouppedResources[ex.Id.ToString()];
                if (!correspondingResource.All(x => x.Lang == "en"))
                    throw new Exception($"Fetching resources has returned mixed languages rather than {"en"} for exercise id: {ex.Id}");

                if (correspondingResource == null)
                    continue;

                ex.ExerciseNames = string.Join(", ", correspondingResource.Select(x=>x.Content));
            }

            var tableRes = new TableResult<TrainingProgramListViewDto>(intermediateResult, totalCount, pageNumber, take);
            return Success(tableRes);
        }

        public async Task<Result<TrainingProgramDto>> Create(TrainingProgramSimplifiedDto dto)
        {
            var trProgDto = _mapper.Map<TrainingProgramDto>(dto);
            trProgDto.GuidId = Guid.NewGuid();
            trProgDto.CreatedAt = DateTime.Now; // create trigger in sql

            var entity = _mapper.Map<TrainingProgramDto, TrainingProgramEntity>(trProgDto, opt => {
                opt.AfterMap((src, dest) => 
                    {
                        dest.User = null;
                        dest.PerformerName = string.Empty; // this will stay here till we delete this column
                    });
            });


            var entry = await _repo.AddAsync(entity);
            return Success(_mapper.Map<TrainingProgramDto>(entry.Entity));
        }

        public async Task<Result<TrainingProgramDto>> Update(TrainingProgramSimplifiedDto dto)
        {
            var existingTrProg = await _repo.GetFirstOrDefaultAsync(x => x.Id == dto.Id, include: TrainingProgramIncludes.FullWithoutUser);
            if (existingTrProg == null)
                return Failure($"There is no training program with id - {dto.Id}");

            existingTrProg.ProgramName = dto.ProgramName;
            existingTrProg.UserId = dto.UserId;
            existingTrProg.Note = dto.Note;
            existingTrProg.UpdatedAt = DateTime.Now;
            existingTrProg.WeeklyPlanDay = dto.WeeklyPlanDay;
            existingTrProg.TotalTrainingsInWeek = dto.TotalTrainingsInWeek;

            var ids = existingTrProg.UnitsOfProgram.Select(x => x.Id).Except(dto.UnitsOfProgram.Select(x => x.Id));

            if (ids != null && ids.Any())
            {
                var toDelete = await _unitRepo.GetAllAsync(x => ids.Contains(x.Id));

                try
                {
                    await _unitRepo.BulkDelete(toDelete);
                }
                catch (Exception e)
                {
                    return Failure($"Something went wrong while deleting Units of training program. Unit ids - {string.Join(',', toDelete.Select(x=>x.Id))}. {e.Message}");
                }
            }

            existingTrProg.UnitsOfProgram = _mapper.Map<List<TrainingProgramUnitEntity>>(dto.UnitsOfProgram);


            var entry = await _repo.UpdateAsync(existingTrProg);
            return Success(_mapper.Map<TrainingProgramDto>(entry.Entity));
        }

        public async Task<Result<TrainingProgramDto>> Delete(long id)
        {
            var obj = await _repo.GetFirstOrDefaultAsync(predicate: x => x.Id == id, include: TrainingProgramIncludes.UnitsOfProgram, disableTracking: true);
            if(obj == null)
                return Failure($"Training program with id {id} not found");

            var entry = await _repo.DeleteAsync(obj);
            return Success(_mapper.Map<TrainingProgramDto>(obj));
        }

        public async Task<Result<TrainingProgramEntity>> GetByGuid(string guid)
        {
            var trProg = await _repo.GetFirstOrDefaultAsync(x=>x.GuidId.ToString() == guid, include: TrainingProgramIncludes.Full);
            if(trProg == null)
                return Failure($"Training program with guid - {guid} not found").ToResult<TrainingProgramEntity>();

            return Success(trProg);
        }

        public async Task<Result<List<TrainingProgramDto>>> GetAll(string q, int? pageNumber, int? take,
            Func<IQueryable<TrainingProgramEntity>, IOrderedQueryable<TrainingProgramEntity>> orderBy = null,
            Func<IQueryable<TrainingProgramEntity>, IIncludableQueryable<TrainingProgramEntity, object>> include = null)
        {
            var query = _repo.GetQueryable(orderBy: orderBy,
                include: include);

            SearchQuery(q, ref query);

            int skip = 0;
            if (pageNumber != null && take != null) 
            { 
                skip = pageNumber.Value * take.Value;

                query = query.Skip(skip).Take(take.Value);
            }


            var trProgs = await query.ToListAsync();
            return Success(_mapper.Map<List<TrainingProgramDto>>(trProgs));
        }

        public async Task<Result<int>> GetAllCount(string q, 
            Func<IQueryable<TrainingProgramEntity>, IIncludableQueryable<TrainingProgramEntity, object>> include = null)
        {
            var query = _repo.GetQueryable(
                include: include);

            SearchQuery(q, ref query);

            var count = await query.CountAsync();
            return Success(count);
        }

        public async Task<Result<TrainingProgramDto>> Duplicate(TrainingProgramDuplicateSimplifiedDto duplicate)
        {
            // Saving new trProg to db
            var trProg = duplicate as TrainingProgramSimplifiedDto;
            trProg.Id = 0;
            var savedTrProgResult = await this.Create(trProg);
            return savedTrProgResult;
        }

        private static void SearchQuery(string q, ref IQueryable<TrainingProgramEntity> query)
        {
            if (!string.IsNullOrEmpty(q))
            {
                q = q.ToLower();
                query = query.Where(x => x.PerformerName.ToLower().Contains(q) || x.User.Username.ToLower().Contains(q) || x.ProgramName.ToLower().Contains(q)); // maybe also first and last names
            }
        }
    }
```

## 4. *Front end*: **1-2 years** of experience (JS, Razor, Vue.JS)

> JS example
``` javascript
async function onDeleteHandler(e){
        debugger;
        let dangerModal = $("#modal-danger");
        let confirmBtn = $("#modal-confirm-btn");
        dangerModal.modal('show');

        // WAIT HERE FOR THE ANSWER
        // Create a promise that resolves when the user confirms or cancels
        const userDecision = new Promise((resolve, reject) => {
            // Handle the "Yes" button click
            confirmBtn.on("click", function() {
                // Unbind the event listener to prevent memory leaks
                confirmBtn.off("click");
                resolve(true);
            });

            // Handle the "Cancel" button click or modal dismissal
            dangerModal.on("hidden.bs.modal", function() {
                // Unbind the event listener to prevent memory leaks
                dangerModal.off("hidden.bs.modal");
                resolve(false);
            });
        });

        const shouldDelete = await userDecision;
        if(!shouldDelete)
            return;

        // fetch api call
        let row = e.target.parentElement.parentElement;

        let smpResId = row.getAttribute('id');
        let testId = row.getAttribute('testId');
        let substance = row.querySelector('td[key="substance"]').textContent;
        let value = row.querySelector('td[key="value"]').textContent;
        
        debugger;

        var requestData = {
            Id: smpResId == '' ? 0 : smpResId, // if edit
            TestId: testId == '' ? 0 : testId,
            Substance: substance,
            Value: value
        };

        const responseData = await deleteTestRes(requestData); //fetch API call    
        if(!responseData){
            dangerModal.modal('hide');
            return;
        }
        
        row.remove();

        dangerModal.modal('hide');
    }
```

## 5. *Unit testing*: **2 years** of experience
> Example of unit testing  (hyrarchy chain resolver). Just a part of it, the original class is 4 times bigger

``` c#
public class InstnTriggerTests
{
    private InstnTrigger instnTrg = new InstnTrigger(
        null, null, Data.MTS.Constants.DataContext.EDIT, null, null, null
    );

    [Fact]
    public void ParentsChain_EmptyInput_ShouldReturnEmptyOutput()
    {
        // Test case data
        // | KeyInstn | KeyInstnParent | TreeValueLeft |
        // |----------|----------------|---------------|
        // Expected return { }

        // Arrange
        var instnsToProcess = new List<InstnTrigger.InstnTreeItem>();
        var heirarchies = new Dictionary<int, List<InstnTrigger.InstnsChianItem>>();

        // Act
        var a = instnTrg.GetAllParentInstns(instnsToProcess, heirarchies);

        // Assert
        Assert.Equal(a, new Dictionary<int, List<int>>(0));
    }

    [Fact]
    public void ParentsChain_SingleInstn_ShouldReturnEmpty()
    {
        // Test case data
        // | KeyInstn | KeyInstnParent | TreeValueLeft |
        // |----------|----------------|---------------|
        // | 1        | NULL           | 1             |
        // Expected return { }

        // Arrange
        var instnsToProcess = new List<InstnTrigger.InstnTreeItem>();
        var heirarchies = new Dictionary<int, List<InstnTrigger.InstnsChianItem>>();

        var treeItem = new InstnTrigger.InstnTreeItem() { KeyInstn = 1, TreeId = 1 }; // Starting from the bottom
        var heirarchyItem = new InstnTrigger.InstnsChianItem() { KeyInstn = 1, KeyInstnParent = null, TreeId = 1, TreeValueLeft = 1 };
        var heirarchyItems = new List<InstnTrigger.InstnsChianItem>() { heirarchyItem };

        instnsToProcess.Add(treeItem);
        heirarchies.Add(heirarchyItem.KeyInstn, heirarchyItems);

        var expected = new Dictionary<int, List<int>>();

        // Act
        var assert = instnTrg.GetAllParentInstns(instnsToProcess, heirarchies);

        // Assert
        Assert.Equal(assert, expected);
    }

    [Fact]
    public void ParentsChain_EveryStepHigherIsAParent_ShouldReturnTheChain()
    {
        // Test case data
        // | KeyInstn | KeyInstnParent | TreeValueLeft |
        // |----------|----------------|---------------|
        // | 1        | NULL           | 1             |
        // | 2        | 1              | 2             |
        // | 3        | 2              | 3             |
        // | 4        | 3              | 4             |
        // | 5        | 4              | 5             |
        // Expected return { 4, 3, 2, 1 }

        // Arrange
        var instnsToProcess = new List<InstnTrigger.InstnTreeItem>();
        var heirarchies = new Dictionary<int, List<InstnTrigger.InstnsChianItem>>();

        var treeItem = new InstnTrigger.InstnTreeItem() { KeyInstn = 5, TreeId = 1 }; // Starting from the bottom

        var heirarchyItem = new InstnTrigger.InstnsChianItem() { KeyInstn = 1, KeyInstnParent = null, TreeId = 1, TreeValueLeft = 1 };
        var heirarchyItem1 = new InstnTrigger.InstnsChianItem() { KeyInstn = 2, KeyInstnParent = 1, TreeId = 1, TreeValueLeft = 2 };
        var heirarchyItem2 = new InstnTrigger.InstnsChianItem() { KeyInstn = 3, KeyInstnParent = 2, TreeId = 1, TreeValueLeft = 3 };
        var heirarchyItem3 = new InstnTrigger.InstnsChianItem() { KeyInstn = 4, KeyInstnParent = 3, TreeId = 1, TreeValueLeft = 4 };
        var heirarchyItem4 = new InstnTrigger.InstnsChianItem() { KeyInstn = 5, KeyInstnParent = 4, TreeId = 1, TreeValueLeft = 5 };

        var heirarchyItems = new List<InstnTrigger.InstnsChianItem>()
        {
            heirarchyItem,
            heirarchyItem1,
            heirarchyItem2,
            heirarchyItem3,
            heirarchyItem4
        };

        instnsToProcess.Add(treeItem);
        heirarchies.Add(heirarchyItem.KeyInstn, heirarchyItems);
        var expected = new Dictionary<int, List<int>>();
        expected.Add(1, new List<int> { 4, 3, 2, 1 });

        // Act
        var assert = instnTrg.GetAllParentInstns(instnsToProcess, heirarchies);

        // Assert
        Assert.Equal(assert, expected);
    }

    [Fact]
    public void ParentsChain_ThereIsAnKeyInstn3ThatIsParentForTwo_ShouldReturnTheChain()
    {
        // Test case data
        // | KeyInstn | KeyInstnParent | TreeValueLeft |
        // |----------|----------------|---------------|
        // | 1        | NULL           | 1             |
        // | 2        | 1              | 2             |
        // | 3        | 2              | 3             |
        // | 4        | 3              | 4             |
        // | 5        | 3              | 5             |
        // Expected return { 3, 2, 1 }

        // Arrange
        var instnsToProcess = new List<InstnTrigger.InstnTreeItem>();
        var heirarchies = new Dictionary<int, List<InstnTrigger.InstnsChianItem>>();

        var treeItem = new InstnTrigger.InstnTreeItem() { KeyInstn = 5, TreeId = 1 }; // Starting from the bottom

        var heirarchyItem = new InstnTrigger.InstnsChianItem() { KeyInstn = 1, KeyInstnParent = null, TreeId = 1, TreeValueLeft = 1 };
        var heirarchyItem1 = new InstnTrigger.InstnsChianItem() { KeyInstn = 2, KeyInstnParent = 1, TreeId = 1, TreeValueLeft = 2 };
        var heirarchyItem2 = new InstnTrigger.InstnsChianItem() { KeyInstn = 3, KeyInstnParent = 2, TreeId = 1, TreeValueLeft = 3 };
        var heirarchyItem3 = new InstnTrigger.InstnsChianItem() { KeyInstn = 4, KeyInstnParent = 3, TreeId = 1, TreeValueLeft = 4 };
        var heirarchyItem4 = new InstnTrigger.InstnsChianItem() { KeyInstn = 5, KeyInstnParent = 3, TreeId = 1, TreeValueLeft = 5 };

        var heirarchyItems = new List<InstnTrigger.InstnsChianItem>()
        {
            heirarchyItem,
            heirarchyItem1,
            heirarchyItem2,
            heirarchyItem3,
            heirarchyItem4
        };

        instnsToProcess.Add(treeItem);
        heirarchies.Add(heirarchyItem.KeyInstn, heirarchyItems);

        var expected = new Dictionary<int, List<int>>();
        expected.Add(1, new List<int> { 3, 2, 1 });

        // Act
        var assert = instnTrg.GetAllParentInstns(instnsToProcess, heirarchies);

        // Assert
        Assert.Equal(assert, expected);
    }
}
```



## 6. *SQL programming*: **5 years** of experience
> Recent example of an query I wrote. Joins and aggregations are not a problem :)
```sql
DECLARE @_id bigint, @_name nvarchar(max), @_notes nvarchar(max);
DECLARE ex_cursor CURSOR FOR 
    SELECT Id, [Name], Notes 
    FROM Exercises;

OPEN ex_cursor;

FETCH NEXT FROM ex_cursor INTO @_id, @_name, @_notes;

WHILE @@FETCH_STATUS = 0
BEGIN
    IF (SELECT COUNT(*) FROM Resources WHERE PrimaryKey = @_id) = 0
    BEGIN
        INSERT INTO Resources (Type, Field, PrimaryKey, Lang, Content)
        VALUES ('exercise', 'name', @_id, 'ru', isnull(@_name, ''));

        INSERT INTO Resources (Type, Field, PrimaryKey, Lang, Content)
        VALUES ('exercise', 'notes', @_id, 'ru', isnull(@_notes, ''));
    END

    FETCH NEXT FROM ex_cursor INTO @_id, @_name, @_notes;
END

CLOSE ex_cursor;  
DEALLOCATE ex_cursor;  
```