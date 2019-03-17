# Ken Shen - Live Project

## Intro
To close out my time at the Tech Academy, I participated in a two week sprint developing a full-scale C# web application in a MVC/MVVM framework with a team of my peers. I had the opportunity to apply my problem solving and root-cause analysis skills towards real-world business needs and complete a number of stories implementing front end, back end, and full stack solutions.

Below are a few of my contributions to the project, as well as my contact details.

### Front End
* [Format and ActionLinks](#format-and-actionlinks)
### Back End
* [Seed Method Update](#seed-method-update)
### Full Stack
* [Index Revision](#index-revision)
### Contact Links
* [Contact](#contact)

## Front End
### Format and ActionLinks
One of our view pages featured a series of misaligned cards that required formatting and actionlinks implemented for each card. I added the matchHeight javascript plugin to the project and functions to streamline the size and height of each card's inner and outer borders. I also added ActionLinks, referencing unique GUIDs for each card, to direct users to an Edit or Delete view page for each given card.

      @model IEnumerable<JobPlacementDashboard.Models.JPBulletin>

      @{
          ViewBag.Title = "Job Bulletins";
      }

      @*While using Bootstrap 3, we're unable to use card styling. To change styling, please
          update Site.css.*@

      <h2>Bulletins</h2>
      <body>
          <div class="row">
              @foreach (var item in Model)
              {
                  <div class="card col-md-3 mb-4 bulletin-container match-height">
                      <div class="card">
                          <p>Category: @Html.DisplayFor(modelItem => item.Bulletin_category)</p>
                          <p>Posted on: @item.Bulletin_date.ToShortDateString()</p>
                          <div class="bulletin-body match-height-2">
                              <p>@Html.DisplayFor(modelItem => item.Bulletin_body)</p>
                              <p>@Html.ActionLink("Edit", "Edit", new { id = item.Bulletin_id })</p>
                              <p>@Html.ActionLink("Delete", "Delete", new { id = item.Bulletin_id })</p>
                          </div>
                      </div>
                  </div>
              }
          </div>
      </body>

      <script type="text/javascript" src="~/Scripts/jquery-3.3.1.min.js"></script>
      <script type="text/javascript" src="~/Scripts/jquery.matchHeight-min.js"></script>
      <script type="text/javascript">
          $(function () {
              var array = $(".match-height");
              var height = 0;
              for (i = 0; i < array.length; i++) {
                  if ($(array[i]).height() > height) {
                      height = $(array[i]).height();
                  }
              }
              $(array).height(height);
          })

          $(function () {
              var array = $(".match-height-2");
              var height = 0;
              for (i = 0; i < array.length; i++) {
                  if ($(array[i]).height() > height) {
                      height = $(array[i]).height();
                  }
              }
              $(array).height(height);
          })
      </script>

## Back End
### Seed Method Update
The live project program used a seeding method to populate a database with multiple tables of sample data for a number of test users. My task was to update the method and populate data columns for any tables that had been omitted from the current iteration of the method. I identified existing code blocks throughout the method that could be quickly renamed and reused to make quick and efficient additions to all required tables with minimum redundancy.
## Full Stack
### Index Revision
My task was to revise an existing Index page from pulling and displaying the entirety of a SQL table's data for each system user, including user GUIDs meant to be kept hidden, to displaying select table data relevant to the Index page for each user, including the user's login name pulled from an adjacent database table.

I first created a new ViewModel for the necessary properties that would be displayed in the revised Index view.

      namespace JobPlacementDashboard.ViewModel
      {
          public class JPAppIndexVM
          {
              [Display(Name = "Student Name")]
              public string StudentName { get; set; }
              [Display(Name = "Company Name")]
              public string CompanyName { get; set; }
              [Display(Name = "Job Title")]
              public string JobTitle { get; set; }
              [Display(Name = "Job Category")]
              public int JobCategory { get; set; }
              [Display(Name = "Company City")]
              public string CompanyCity { get; set; }
              [Display(Name = "State Code")]
              public string StateCode { get; set; }
              [Display(Name = "Application Date")]
              public DateTime ApplicationDate { get; set; }
              [Display(Name = "Heard Back")]
              public bool HeardBack { get; set; }
              public bool Interview { get; set; }
          }
      }

I then revised the Index page's associated controller action to reference any SQL data tables needed to populate and return a list of ViewModels.

        private ApplicationDbContext db = new ApplicationDbContext();

        // GET: JPApplications
        public ActionResult Index()
        {
            if (User.IsInRole("Admin"))
            {
                IEnumerable<ApplicationUser> users = db.Users.ToList();

                var apps = db.JPApplications.ToList();
                var students = db.JPStudents.ToList();
                var appIndexVMs = new List<JPAppIndexVM>();
                foreach (var student in students)
                {
                    foreach (var app in apps)
                    {
                        var appID = app.Application_id;
                        if (app.Student_id == student.Student_id)
                        {
                            appIndexVMs.Add(new JPAppIndexVM()
                            {
                                StudentName = users.Where(x => x.Id == student.ApplicationUser.Id).FirstOrDefault().UserName,
                                CompanyName = db.JPApplications.Where(x => x.Student_id == student.Student_id && x.Application_id ==  appID).FirstOrDefault().Company_name,
                                JobTitle = db.JPApplications.Where(x => x.Student_id == student.Student_id && x.Application_id == appID).FirstOrDefault().Job_title,
                                JobCategory = db.JPApplications.Where(x => x.Student_id == student.Student_id && x.Application_id == appID).FirstOrDefault().JPJob_catagory,
                                CompanyCity = db.JPApplications.Where(x => x.Student_id == student.Student_id && x.Application_id == appID).FirstOrDefault().Company_city,
                                StateCode = db.JPApplications.Where(x => x.Student_id == student.Student_id && x.Application_id == appID).FirstOrDefault().State_code,
                                ApplicationDate = db.JPApplications.Where(x => x.Student_id == student.Student_id && x.Application_id == appID).FirstOrDefault().Application_date,
                                HeardBack = db.JPApplications.Where(x => x.Student_id == student.Student_id && x.Application_id == appID).FirstOrDefault().Heard_back,
                                Interview = db.JPApplications.Where(x => x.Student_id == student.Student_id && x.Application_id == appID).FirstOrDefault().Interview
                            });
                        }
                    }
                }
                return View(appIndexVMs);
            }
            else return View("~/Views/Home/Index.cshtml");
        }

Finally, I updated the existing Index view page for a structured display of ViewModel properties for each user in the database.

    @using JobPlacementDashboard.ViewModel
    @model List<JPAppIndexVM>

    @{
        ViewBag.Title = "Index";
    }

    @Html.DisplayNameFor(model => model)
    <h2>Index</h2>

    <p>
        @Html.ActionLink("Create New", "Create")
    </p>
    <table class="table">
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.FirstOrDefault().StudentName)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.FirstOrDefault().CompanyName)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.FirstOrDefault().JobTitle)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.FirstOrDefault().JobCategory)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.FirstOrDefault().CompanyCity)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.FirstOrDefault().StateCode)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.FirstOrDefault().ApplicationDate)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.FirstOrDefault().HeardBack)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.FirstOrDefault().Interview)
            </th>
            <th></th>
        </tr>

        @foreach (var item in Model)
        {
            <tr>
                <td>
                    @Html.DisplayFor(modelItem => item.StudentName)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.CompanyName)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.JobTitle)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.JobCategory)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.CompanyCity)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.StateCode)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.ApplicationDate)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.HeardBack)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.Interview)
                </td>
            </tr>
        }

    </table>

## Contact
kshen92@gmail.com <br>
[Github](https://github.com/KShen92) <br>
[LinkedIn](https://www.linkedin.com/in/kenneth-shen-4b1728b4)
