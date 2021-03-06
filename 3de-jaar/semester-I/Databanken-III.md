---
title: Databanken III
link: http://robinmalfait.com/3de-jaar/semester-I/Databanken-III.md
---

[Chamilo](https://chamilo.hogent.be/index.php?application=Chamilo%5CApplication%5CWeblcms&go=CourseViewer&course=24073)


# PL/SQL

## Slides

| pdf           | keywords                |
| ------------- | ----------------------- |
| PLSQL_s01.pdf | `introduction`, `benefits`, `creating PL/SQL blocks` |
| PLSQL_s02.pdf | `variables`, `types`, `%TYPE`, `functions`, `implicit/explicit data conversion`, `nested blocks`, `variable scope`, `<<outer>>`, `Good Programming Practices (¯\_(ツ)_/¯)`, `naming_conventions v_ (variable), c_ (constant), p_ (parameter)` |
| PLSQL_s03.pdf | `retrieving data`, `naming conventions`, `manipulating data`, `implicit cursors`, `transactional control statements` |
| PLSQL_s04.pdf | `Conditionals`, `IF`, `THEN`, `ELSE`, `ELSIF`, `CASE`, `basic loops`, `WHILE`, `FOR`, `nested loops`, `loop labels`, `<<outer_loop>>`, `<<inner_loop>>` |
| PLSQL_s05.pdf | `Explicit cursors`, `DECLARE`, `OPEN`, `FETCH`, `CLOSE`, `attributes`, `records`, `%ROWTYPE`, `%ISOPEN`, `%NOTFOUND`, `%FOUND`, `%ROWCOUNT`, `cursor FOR loops`, `cursor with parameters`, `cursors FOR UPDATE`, `NOWAIT`, `FOR UPDATE OF column-name`, `WHERE CURRENT OF cursor-name`, `multiple cursors` |
| PLSQL_s06.pdf | `User-Defined Records (custom types)`, `Indexing Tables of Records` |
| PLSQL_s07.pdf | `Handling exceptions`, `EXCEPTION`, `WHEN`, `NO_DATA_FOUND`, `TOO_MANY_ROWS`, `OTHERS`, `1. Predefined Oracle server error`, `2. Non-predefiend Oracale server error`, `3. User-defined error`, `INVALID_CURSOR`, `ZERO_DIVIDE`, `DUP_VAL_ON_INDEX`, `slide 32`, `SQLCODE`, `SQLERRM`, `slide 42`, `RAISE`, `RAISE_APPLICATION_ERROR`, `scope of exceptions` |
| PLSQL_s08.pdf | `creating PROCEDURE`, `PROCEDURE`, `CREATE OR REPLACE PROCEDURE xxx IS`, `parameters`, `DESCRIBE (slide 29)`, `IN`, `OUT`, `IN OUT` |
| PLSQL_s09.pdf | `Creating FUNCTION`, `FUNCTION`, `CREATE OR REPLACE FUNCTION xxx IS`, `RETURN`, `USER`, `SYSDATE`, `IS`, `AS`, `Object Privilege (slide 38)`, `AUTHID`, `CURRENT_USER` |
| PLSQL_s10.pdf | `creating PACKAGE`, `PACKAGE`, `CREATE OR REPLACE PACKAGE xxx IS`, `Package Body`, `CREATE OR REPLACE PACKAGE BODY xxx IS`, `DESCRIBE` |
| PLSQL_s11.pdf | `improving performance`, `NOCOPY`, `DETERMINISTIC`, `FORALL`, `BULK COLLECT`, `FETCH`, `RETURNING`, `SQL%BULK_ROWCOUNT(i)`, `SAVE EXCEPTIONS`, `SQL%BULK_EXCEPTIONS.COUNT`, `SQL%BULK_EXCEPTIONS(i).ERROR_INDEX`, `SQL%BULK_EXCEPTIONS(i).ERROR_CODE` |
| PLSQL_s12.pdf | `dynamic sql`, `EXEXECUTE IMMEDIATE xxx`, `INTO`, `USING`, `dynamic_string`, `define_variable`, `record`, `bind_argument`, `ALTER` |

## Exceptions

```plsql
/*
(1) Write a program that writes out the firstname, lastname and salary of the CEO.
Catch any eexceptions (e.g. no CEO / multiple CEO's / ...)

See slide 28
*/

DECLARE
  v_first_name employees.firstname%TYPE;
  v_last_name employees.lastname%TYPE;
  v_salary employees.salary%TYPE;
BEGIN
  SELECT firstname, lastname, salary
  INTO v_first_name, v_last_name, v_salary
  FROM employees
  WHERE service = 'CEO';
  /*where service = 'CFO'  -- NO_DATA_FOUND*/
  /*where service = 'TRAINER' -- TOO_MANY_ROWS*/
  /*where service = 1 -- OTHERS*/

  DBMS_OUTPUT.PUT_LINE('CEO: ' || v_first_name || ' ' || v_last_name || ' has a salary of ' || v_salary);

  EXCEPTION
    WHEN TOO_MANY_ROWS THEN
      DBMS_OUTPUT.PUT_LINE('There are multiple CEOs');
    WHEN NO_DATA_FOUND THEN
      DBMS_OUTPUT.PUT_LINE('There is no CEO');
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Another type of error occured with message: ' || sqlerrm || ' and code ' || sqlcode);
END;


/*
(2) Department id opgegeven.
Als department niet bestaat --> user defined exception
Anders: # werknemers voor department

Slide 52
*/

DECLARE
  v_department_id departments.departmentid%TYPE;

  e_non_existing_department EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_non_existing_department, -20001);

  v_count_departments NUMBER(5, 0) := 0;
BEGIN
  v_department_id := '&Give_the_department_id';

  SELECT count(departmentid)
  INTO v_count_departments
  FROM departments
  WHERE departmentid = v_department_id;

  IF v_count_departments = 0 THEN
    RAISE_APPLICATION_ERROR(-20001, 'No such departments');
  END IF;

  EXCEPTION
    WHEN e_non_existing_department THEN
      DBMS_OUTPUT.PUT_LINE('SQLERRM: ' || SQLERRM || ' SQLCODE ' || SQLCODE);
END;
```

## Procedures

```plsql
/*
(1) Write a procedure that shows the data (firstname + lastname + service) from all the employees on a given location.
Use a cursor to loop through the data. Throw a user defined exception if the location doesn't exist
*/

CREATE OR REPLACE PROCEDURE showMeh(p_location varchar2) IS

 CURSOR employee_cursor IS
  SELECT firstname, lastname, service
  FROM employees
  INNER JOIN departments ON departments.departmentid = employees.department
  WHERE location = p_location;

  v_count number := 0;

  e_non_existing_location EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_non_existing_location, -20001);

BEGIN

  SELECT count(location)
  INTO v_count
  FROM departments
  WHERE location = p_location;

  IF v_count = 0
  THEN
    RAISE_APPLICATION_ERROR(-20001, 'Location does not exist');
  END IF;

  FOR v_employee_record IN employee_cursor
  LOOP
    DBMS_OUTPUT.PUT_LINE(v_employee_record.firstname || ' ' || v_employee_record.lastname || ' - ' || v_employee_record.service);
  END LOOP;

  EXCEPTION
    WHEN e_non_existing_location THEN
      DBMS_OUTPUT.PUT_LINE('SQLERRM: ' || SQLERRM || ' SQLCODE ' || SQLCODE);
END;


/* TEST */
BEGIN
  DBMS_OUTPUT.PUT_LINE('TEST FOR BLABLA');
  DBMS_OUTPUT.PUT_LINE('');
  showMeh('BLABLA');

  DBMS_OUTPUT.PUT_LINE('');
  DBMS_OUTPUT.PUT_LINE('');

  DBMS_OUTPUT.PUT_LINE('TEST FOR ANTWERP');
  DBMS_OUTPUT.PUT_LINE('');
  showMeh('ANTWERP');
END;
```

## Functions

```plsql
/*
(1) The function checkService has a parameter p_service and returns true if the service exists,
otherwise false + write a test program
*/

CREATE OR REPlACE FUNCTION checkService (p_service employees.service%TYPE)
RETURN boolean IS
  total number := 0;
BEGIN

  SELECT count(*)
  INTO total
  FROM employees
  WHERE service = p_service;

  return total <> 0;
END;

BEGIN
  IF checkService('TRAINER') THEN
    DBMS_OUTPUT.PUT_LINE('Service TRAINER exists');
  ELSE
    DBMS_OUTPUT.PUT_LINE('Service TRAINER does not exists');
  END IF;
END;

/*
(2) The function checkSalary has a parameter p_service and p_salary and returns true
if the salary is in a 15% range of the average salary for this service.
Otherwise the function returns false. + write a test program
*/

CREATE OR REPlACE FUNCTION checkSalary (p_service employees.service%TYPE, p_salary employees.salary%TYPE)
RETURN boolean IS
  average number := 0;
  upperBound number := 0;
  lowerBound number := 0;
BEGIN

  SELECT avg(salary)
  INTO average
  FROM employees
  WHERE service = p_service;

  upperBound := average * 1.15;
  lowerBound := average - (average * .15);

  return (p_salary <= upperBound) AND (p_salary >= lowerBound);
END;

BEGIN
  IF checkSalary('TRAINER', 1970) THEN
    DBMS_OUTPUT.PUT_LINE('Salary is in a 15% range');
  ELSE
    DBMS_OUTPUT.PUT_LINE('Salary is not in a 15% range');
  END IF;
END;
```

## Package

```plsql
CREATE OR REPLACE PACKAGE dbvideos_package1 IS

TYPE film_rec_type IS RECORD (
  bandcodee films.bandcode%TYPE,
  titel films.titel%TYPE,
  genre genres.genre%TYPE,
  prijs films.prijs%TYPE,
  aantalkeerverhuurd films.totverhuurd%TYPE
);

TYPE film_tab_type IS TABLE OF film_rec_type INDEX BY BINARY_INTEGER;

PROCEDURE films_per_genre;

FUNCTION geef_pop_films_per_leeftcat (p_min_leeftijd NUMBER, p_max_leeftijd NUMBER, p_aantal NUMBER) RETURN film_tab_type;

FUNCTION wijzig_prijs (p_percentage NUMBER, p_maatschappij maatschappijen.maatschappijnaam%TYPE) RETURN NUMBER;

e_video_exception EXCEPTION;
PRAGMA EXCEPTION_INIT(e_video_exception, -20001);

END dbvideos_package1;
```

```plsql
CREATE OR REPLACE PACKAGE BODY dbvideos_package1 IS

PROCEDURE films_per_genre IS
CURSOR c_genres IS SELECT * FROM genres;
CURSOR c_films (p_genrecode films.genrecode%TYPE) IS SELECT bandcode, titel FROM films WHERE genrecode = p_genrecode;
BEGIN
  FOR v_genrerecord IN c_genres LOOP
    DBMS_OUTPUT.PUT_LINE('Genre ' || v_genrerecord.genre);

    FOR v_filmrec IN c_films (v_genrerecord.genrecode) LOOP
      DBMS_OUTPUT.PUT_LINE(v_filmrec.bandcode || ' ' || v_filmrec.titel);
    END LOOP;
  END LOOP;
END;

FUNCTION geef_pop_films_per_leeftcat (p_min_leeftijd NUMBER, p_max_leeftijd NUMBER, p_aantal NUMBER) RETURN film_tab_type IS
  v_result film_tab_type;
BEGIN
  SELECT f.bandcode, f.titel, g.genre, f.prijs, count(v.bandcode)
  BULK COLLECT INTO v_result
  FROM films f JOIN genres g ON f.genrecode = g.genrecode
  JOIN verhuur v ON v.bandcode = f.bandcode
  JOIN klantenvideo k ON k.klantcode = v.klantcode
  WHERE TO_NUMBER(TO_CHAR(SYSDATE, 'YYYY')) - TO_NUMBER(TO_CHAR(k.geboortedatum, 'YYYY')) BETWEEN p_min_leeftijd AND p_max_leeftijd
  AND ROWNUM <= p_aantal
  GROUP BY f.bandcode, f.titel, g.genre, f.prijs
  ORDER BY count(v.bandcode) DESC;

  RETURN v_result;
END;

FUNCTION wijzig_prijs (p_percentage NUMBER, p_maatschappij maatschappijen.maatschappijnaam%TYPE) RETURN NUMBER IS
  v_count NUMBER;
  TYPE bandcodes_tab_type IS TABLE OF films.bandcode%TYPE INDEX BY BINARY_INTEGER;
  v_bandcodes bandcodes_tab_type;
BEGIN
  SELECT count(maatschappijcode) INTO v_count
  FROM maatschappijen
  WHERE maatschappijnaam = p_maatschappij;
  IF v_count = 0 THEN
    RAISE_APPLICATION_ERROR(-20001, 'De maatschappij bestaat niet');
  END IF;
  SELECT bandcode BULK COLLECT INTO v_bandcodes
  FROM films f JOIN maatschappijen m
  ON f.maatschappijcode = m.maatschappijcode
  WHERE m.maatschappijnaam = p_maatschappij;

  FORALL i IN v_bandcodes.FIRST .. v_bandcodes.LAST
    UPDATE films
    SET prijs = prijs * (1 + p_percentage)
    WHERE bandcode = v_bandcodes(i);

  EXCEPTION
    WHEN e_video_exception THEN
      DBMS_OUTPUT.PUT_LINE('SQLCODE ' || ' ' || SQLCODE);
    WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('SQLCODE ' || ' ' || SQLCODE);
END;

END dbvideos_package1;
```

## Afsluitende herhalings oefeningen

```plsql
/********
Oefening 1
********/

/* Declaratie van de package specification */
CREATE OR REPLACE PACKAGE dbvideos_package1 IS
	TYPE film_rec_type IS RECORD (
		bandcode films.bandcode%TYPE,
		titel	films.titel%TYPE,
		genre	genres.genre%TYPE,
		prijs	films.prijs%TYPE,
		aantalkeerverhuurd films.verhuurd%TYPE
	);
	TYPE film_tab_type IS TABLE OF film_rec_type
	INDEX BY BINARY_INTEGER;

	e_video_exception	EXCEPTION;
	PRAGMA EXCEPTION_INIT(e_video_exception, -20001);

	procedure films_per_genre;
	function geef_pop_films_per_leeftcat (p_min_leeftijd NUMBER,
		p_max_leeftijd NUMBER, p_aantal NUMBER) RETURN film_tab_type;
	function wijzig_prijs (p_percentage NUMBER, p_maatschappij
	maatschappijen.maatschappijnaam%TYPE) RETURN NUMBER;

END dbvideos_package1;


/* Declaratie van de package body */

CREATE OR REPLACE PACKAGE BODY dbvideos_package1 IS

	procedure films_per_genre IS
		CURSOR c_genres IS SELECT * FROM genres;
		CURSOR c_films (p_genrecode genres.genrecode%TYPE) IS
		SELECT bandcode, titel FROM films WHERE genrecode = p_genrecode;
		BEGIN
			FOR v_genrerec IN c_genres LOOP
				DBMS_OUTPUT.PUT_LINE('Genre ' || v_genrerec.genre);
				FOR v_filmrec IN c_films (v_genrerec.genrecode)
				LOOP
				DBMS_OUTPUT.PUT_LINE(v_filmrec.bandcode ||
					' ' || v_filmrec.titel);
				END LOOP;
			END LOOP;
		END;

function geef_pop_films_per_leeftcat (p_min_leeftijd NUMBER,
		p_max_leeftijd NUMBER, p_aantal NUMBER) RETURN film_tab_type IS
	v_result film_tab_type;

BEGIN
	SELECT f.bandcode, f.titel, g.genre, f.prijs, COUNT(v.bandcode)
	BULK COLLECT INTO v_result
	FROM films f JOIN genres g ON f.genrecode = g.genrecode
	JOIN verhuur v ON v.bandcode = f.bandcode
	JOIN klantenvideo k ON v.klantcode = k.klantcode
	WHERE TO_NUMBER(TO_CHAR(SYSDATE,'YYYY')) -
		TO_NUMBER(TO_CHAR(k.geboortedatum, 'YYYY'))
		BETWEEN p_min_leeftijd AND p_max_leeftijd
	AND WHERE rownum <= p_aantal
	GROUP BY f.bandcode, f.titel, g.genre, f.prijs
	ORDER BY COUNT(v.bandcode) DESC;

	RETURN v_result;
END;

function wijzig_prijs (p_percentage NUMBER, p_maatschappij
	maatschappijen.maatschappijnaam%TYPE) RETURN NUMBER IS
	TYPE bandcodes_tab_type IS TABLE OF films.bandcode%TYPE
	INDEX BY BINARY_INTEGER;
	v_bandcodes_tab	bandcodes_tab_type;
	v_count	NUMBER;

BEGIN
	SELECT COUNT(maatschappijcode) INTO v_count
	FROM maatschappijen
	WHERE maatschappijnaam = p_maatschappij;
	IF v_count = 0 THEN
		RAISE_APPLICATION_ERROR(-20001, 'Maatschappij doesn''t exist');
	END IF;
	SELECT f.bandcode BULK COLLECT INTO v_bandcodes_tab
	FROM films f JOIN maatschappijen m
	ON f.maatschappijcode = m.maatschappijcode
	WHERE m.maatschappijnaam = p_maatschappij;

	FORALL i in v_bandcodes_tab.FIRST .. v_bandcodes_tab.LAST
		UPDATE films
		SET prijs = prijs * (1 + p_percentage)
		WHERE bandcode = v_bandcodes_tab(i);

	RETURN v_bandcodes_tab.COUNT();
END;




END dbvideos_package1;


/* Testcode */
DECLARE
	v_film_tab	dbvideos_package1.film_tab_type;
	v_count	NUMBER;

BEGIN
	dbvideos_package1.films_per_genre();
	v_film_tab := dbvideos_package1.geef_pop_films_per_leeftcat(20, 25,5);
	FOR i in v_film_tab.FIRST .. v_film_tab.LAST LOOP
		IF v_film_tab.EXISTS(i) THEN
			DBMS_OUTPUT.PUT_LINE(v_film_tab(i).titel || ' '
			|| v_film_tab(i).genre);
		END IF;
	END LOOP;
	v_count := dbvideos_package1.wijzig_prijs(0.05, 'HOLLYWOOD VIDEO');
	DBMS_OUTPUT.PUT_LINE('aantal gewijzigde records is ' || v_count);
END;




/**********
Oefening 2
**********/



/* Declaratie van de package specification */
create or replace package dbvideos_package2  IS
TYPE genre_rec_type IS RECORD (genrecode genres.genrecode%TYPE, genre genres.genre%TYPE,
                              aantal INTEGER);
TYPE genre_tab_type is TABLE OF genre_rec_type INDEX BY PLS_INTEGER;
  e_video_exception EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_video_exception, -20001);

procedure films_per_datum;
function geef_genres_per_klant (p_klantcode klantenvideo.klantcode%TYPE) return genre_tab_type;
function wijzig_aantal_exemplaren(p_maximum NUMBER) return NUMBER;
END VideoPackage;
create or replace package BODY dbvideos_package2 IS
procedure films_per_datum as
cursor cur_datum is select distinct verhuurdatum from verhuur order by verhuurdatum;
cursor cur_films (p_verhuurdatum verhuur.verhuurdatum%type) is
   select distinct f.bandcode, f.titel, g.genre from films f join genres g
   on g.genrecode = f.genrecode join verhuur v on f.bandcode = v.bandcode
   where v.verhuurdatum = p_verhuurdatum
   order by f.titel;
begin
  for datum_rec in cur_datum loop
    dbms_output.put_line('Datum ' || datum_rec.verhuurdatum);
    for film_rec in cur_films(datum_rec.verhuurdatum) loop
      dbms_output.put_line(film_rec.bandcode || ' ' || film_rec.titel);
    end loop;
      dbms_output.put_line('');
  end loop;
END;

function geef_genres_per_klant (p_klantcode klantenvideo.klantcode%TYPE) return genre_tab_type IS
v_resultaat genre_tab_type;
  v_aantal NUMBER(6,0);
  v_teller NUMBER(6,0);
cursor cur_genres is
   select distinct g.genrecode, g.genre, count(v.bandcode) As aantal from films f join genres g
   on g.genrecode = f.genrecode join verhuur v on f.bandcode = v.bandcode
   where v.klantcode = p_klantcode
   group by g.genrecode, g.genre;
BEGIN
  SELECT count(klantcode) into v_aantal from klantenvideo where klantcode = p_klantcode;
  if v_aantal = 0 then
      RAISE_APPLICATION_ERROR(-20001, 'Er is iets fout met de maatschappijnaam');
  end if;

  v_teller := 1;
  for genre_rec in cur_genres loop
    v_resultaat(v_teller).genrecode := genre_rec.genrecode;
    v_resultaat(v_teller).genre := genre_rec.genre;
    v_resultaat(v_teller).aantal := genre_rec.aantal;  
    v_teller := v_teller + 1;
  end loop;

  return v_resultaat;
EXCEPTION
WHEN  e_video_exception then
dbms_output.put_line (SQLERRM);
WHEN OTHERS THEN
dbms_output.put_line ('Er is een fout opgetreden!');

END;

function wijzig_aantal_exemplaren(p_maximum NUMBER) return NUMBER IS
TYPE t_films IS TABLE OF films.bandcode%TYPE INDEX BY BINARY_INTEGER;
  v_films_tab t_films;

BEGIN
  SELECT bandcode BULK COLLECT INTO v_films_tab FROM films
  WHERE voorraad + verhuurd < p_maximum;

  FORALL i IN v_films_tab.FIRST..v_films_tab.LAST
  UPDATE films
  SET voorraad = p_maximum - verhuurd
  WHERE bandcode = v_films_tab(i);

  return v_films_tab.count();
  END;


end videopackage;


/* Testprogramma */

DECLARE
v_aantal NUMBER(6,0) := 0;
v_resultaat videopackage.genre_tab_type;
begin
videopackage.films_per_datum();
v_resultaat := videopackage.geef_genres_per_klant(6);
  FOR i IN v_resultaat.FIRST..v_resultaat.LAST LOOP
      IF v_resultaat.EXISTS(i) THEN
    DBMS_OUTPUT.PUT_LINE(v_resultaat(i).genrecode || ' ' || v_resultaat(i).genre || ' - ' || v_resultaat(i).aantal);
  END IF;
  END LOOP;
  dbms_output.put_line('');  
  v_aantal := videopackage.wijzig_aantal_exemplaren(5);
  dbms_output.put_line('aantal gewijzigde records is ' || v_aantal);

end;
```
