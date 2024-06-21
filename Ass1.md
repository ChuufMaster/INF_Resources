
# ASS1_Back

```cs
using Architecture.Models;
using Architecture.ViewModel;
using Microsoft.AspNetCore.Components.Forms;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using System.Reflection.Metadata.Ecma335;

namespace Architecture.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class CourseController : ControllerBase
    {
        private readonly ICourseRepository _courseRepository;

        public CourseController(ICourseRepository courseRepository)
        {
            _courseRepository = courseRepository;
        }

        [HttpGet]
        [Route("GetAllCourses")] //returns a list of courses
        public async Task<IActionResult> GetAllCourses()
        {
            try
            {
                var results = await _courseRepository.GetAllCourseAsync();
                return Ok(results);
            }
            catch (Exception)
            {
                return StatusCode(500,"Internal Server Error. Please contact support.");
            }
        }

        [HttpGet]
        [Route("GetCourse/{courseId}")] //returns a specific course 
        public async Task<IActionResult> GetCourseAsync(int courseId)
        {
            try
            {
                var results = await _courseRepository.GetCourseAsync(courseId);

                if (results == null) return NotFound("Course does not exist");

                return Ok(results);
            }
            catch (Exception)
            {
                return StatusCode(500, "Internal Server Error. Please contact support.");
            }
        }

        [HttpPost]
        [Route("AddCourse")]
        public async Task <IActionResult> AddCourse(CourseViewModel cvm) 
        {
            var course = new Course { Name = cvm.Name, Duration = cvm.Duration, Description = cvm.Description };

            try
            {
                _courseRepository.Add(course);
                await _courseRepository.SaveChangesAsync();
            }
            catch (Exception)
            {
                return BadRequest("Invalid transaction");
            }

            return Ok(course);
        }

        [HttpPut]
        [Route("EditCourse/{courseId}")]
        public async Task<ActionResult<CourseViewModel>> EditCourse(int courseId, CourseViewModel courseModel) 
        {
            try
            {
                var existingCourse = await _courseRepository.GetCourseAsync(courseId);
                if (existingCourse == null) return NotFound($"The course does not exist");

                existingCourse.Name = courseModel.Name;
                existingCourse.Duration = courseModel.Duration;
                existingCourse.Description = courseModel.Description;

                if (await _courseRepository.SaveChangesAsync())
                {
                    return Ok(existingCourse);
                }

            }
            catch (Exception)
            {
                return StatusCode(500, "Internal Server Error. Please contact support.");
                
            }
            return BadRequest("Your request is invalid.");

           
        }

        [HttpDelete]
        [Route("DeleteCourse/{courseId}")]
        public async Task<IActionResult> DeleteCourse(int courseId)
        {
            try
            {
                var existingCourse = await _courseRepository.GetCourseAsync(courseId);
                if (existingCourse == null) return NotFound($"The course does not exist");
                _courseRepository.Delete(existingCourse);

                if(await _courseRepository.SaveChangesAsync()) return Ok(existingCourse);
            }
            catch (Exception)
            {

                return StatusCode(500, "Internal Server Error. Please contact support.");
            }

            return BadRequest("Your request is invalid");
        }
    }
}
```

# ASS1_Front

## Add Course

```ts

import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { DataService } from '../services/data.service';
import { Course } from '../shared/course';

@Component({
  selector: 'app-add-course',
  templateUrl: './add-course.component.html',
  styleUrls: ['./add-course.component.scss']
})
export class AddCourseComponent implements OnInit {

  addCourseAtt: Course = {
    courseId: 0,
    name: '',
    duration: '',
    description: '',
  };

  constructor(private dataService: DataService, private router: Router ) { }

  ngOnInit(): void {
  }

  addCourse(){
    this.dataService.addCourse(this.addCourseAtt).subscribe({
      next: (course) => {
        this.router.navigate(['courses'])
        console.log(course)

      }
    });
  }


  cancel(){
    this.router.navigate(["courses"]);
  };
}
```

## Courses Component

```ts

import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { DataService } from '../services/data.service';
import { Course } from '../shared/course';


@Component({
  selector: 'app-courses',
  templateUrl: './courses.component.html',
  styleUrls: ['./courses.component.scss']
})
export class CoursesComponent implements OnInit {
  
  courses:Course[] = []

  constructor(private dataService: DataService, private router: Router) { }

  ngOnInit(): void {
    this.GetCourses()
    
    console.log(this.courses)
  }

  GetCourses()
  {
    this.dataService.GetCourses().subscribe(result => {
      let courseList:any[] = result
      courseList.forEach((element) => {
        this.courses.push(element)
      });
      this.courses.reverse();
    })
  }

  deleteCourse(id:number){
    this.dataService.deleteCourse(id).subscribe({
      next: (response) => {
        alert("Deleted");
        // this.GetCourses();
        window.location.reload();
      }
    });
  }

  cancel(){
    this.router.navigate(["courses"]);
  };
}
```

## Edit Course

```ts

import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { DataService } from '../services/data.service';
import { Course } from '../shared/course';

@Component({
  selector: 'app-edit-course',
  templateUrl: './edit-course.component.html',
  styleUrls: ['./edit-course.component.scss']
})
export class EditCourseComponent implements OnInit {

  courseAtt: Course = {
    courseId: 0,
    name: '',
    duration: '',
    description: ''
  };


  constructor(private route: ActivatedRoute, private dataService: DataService, private router: Router ) { }

  ngOnInit(): void {
    this.route.paramMap.subscribe({
      next: (params) => {
        const courseId = params.get('courseId');

        //Call the API
        if(courseId){
          this.dataService.getCourseId(courseId).subscribe({
            next: (response) => {
              this.courseAtt = response;
            }
          });
        }
      }
    })
  }


  updateCourse(){
    this.dataService.updateEmployee(this.courseAtt.courseId, this.courseAtt).subscribe({
      next: (response) =>{
        this.router.navigate(['courses'])
      }
    });
 
  }

  cancel(){
    this.router.navigate(["courses"]);
  };
}
```

## Data Service

```ts

import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { map, Observable, Subject } from 'rxjs';
import { Course } from '../shared/course';

@Injectable({
  providedIn: 'root'
})
export class DataService {

  apiUrl = 'http://localhost:5116/api/'

  httpOptions ={
    headers: new HttpHeaders({
      ContentType: 'application/json'
    })
  }

  constructor(private httpClient: HttpClient) { 
  }

  GetCourses(): Observable<any>{
    return this.httpClient.get(`${this.apiUrl}Course/GetAllCourses`)
    .pipe(map(result => result))
  }

  addCourse(addCourseAtt: Course){
    return this.httpClient.post<Course>(`${this.apiUrl}Course/AddCourse`, addCourseAtt)
    .pipe(map(result => result))
  }

  getCourseId(courseId: string): Observable<Course>{
    return this.httpClient.get<Course>(`${this.apiUrl}Course/GetCourse/` + courseId)
  }


  updateEmployee(id: number, courseAtt: Course): Observable<Course>{
    return this.httpClient.put<Course>(`${this.apiUrl}Course/EditCourse/` + id, courseAtt)
  }

  deleteCourse(courseId: number): Observable<Course>{
    return this.httpClient.delete<Course>(`${this.apiUrl}Course/DeleteCourse/` + courseId)
  }
}


```

## Routing

```ts

import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { CoursesComponent } from './course/courses.component';
import { AddCourseComponent } from './add-course/add-course.component';
import { EditCourseComponent } from './edit-course/edit-course.component';

const routes: Routes = [
  
  {path: 'courses', component: CoursesComponent},  
  {path: '', redirectTo: '/courses', pathMatch: 'full'},
  {path: 'addCourses', component: AddCourseComponent},
  {path: 'editCourses/:courseId', component: EditCourseComponent}
  
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```
