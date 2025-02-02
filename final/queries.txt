SELECT name, species
FROM animals
WHERE name LIKE 'А%';

SELECT d.name, count(*)
FROM departments AS d 
INNER JOIN animals AS a ON d.department_id = a.department_id
GROUP BY d.name
HAVING d.name = 'Приматы';

WITH herbivore_workers AS (
SELECT worker_id, w.name
FROM departments AS dep 
INNER JOIN directors AS dir ON dep.department_id = dir.department_id
LEFT JOIN workers AS w ON dir.director_id = w.director_id
WHERE dep.name = 'Крупные травоядные' )

SELECT hw.name, t.title, a.name, a.species
FROM herbivore_workers AS hw
INNER JOIN tasks AS t ON hw.worker_id = t.worker_id
INNER JOIN animals AS a ON t.animal_id = a.animal_id;


CREATE OR REPLACE VIEW number_workers AS (
 SELECT dep.name AS name, wd.number AS number 
 FROM departments AS dep INNER JOIN
  (SELECT d.department_id, count(*) AS number
   FROM directors AS d
   INNER JOIN workers AS w ON d.director_id = w.director_id
   GROUP BY d.director_id ) AS wd
 ON dep.department_id = wd.department_id );

SELECT * FROM number_workers;

SELECT 
CASE WHEN (SELECT number FROM number_workers AS nw WHERE nw.name = 'Мелкие млекопитающие') > (SELECT number FROM number_workers  AS nw WHERE nw.name = 'Крупные травоядные') THEN 'Мелкие млекопитающие'
ELSE 'Крупные травоядные'
END as department;

SELECT dep.name, dep.costs AS fixed_costs, 
(CASE WHEN ww.workers_wage IS NULL AND fc.food_costs IS NULL THEN 0.00
      WHEN ww.workers_wage IS NULL THEN fc.food_costs
	WHEN fc.food_costs IS NULL THEN ww.workers_wage
      ELSE ww.workers_wage + fc.food_costs
END) AS variable_costs,
(CASE WHEN ww.workers_wage IS NULL AND fc.food_costs IS NULL THEN  dep.costs
      WHEN ww.workers_wage IS NULL THEN dep.costs + fc.food_costs
	WHEN fc.food_costs IS NULL THEN dep.costs + ww.workers_wage
      ELSE dep.costs + ww.workers_wage + fc.food_costs
END) AS total_costs
FROM
  (SELECT d.department_id, sum(w.wage) AS workers_wage
   FROM directors AS d
   LEFT JOIN workers AS w ON d.director_id = w.director_id
   GROUP BY d.department_id) AS ww
RIGHT JOIN departments AS dep 
ON dep.department_id = ww.department_id
LEFT JOIN
  (SELECT a.department_id, sum(f.price * a.food_weight) AS food_costs
   FROM animals AS a
   INNER JOIN food AS f ON a.food_id = f.food_id
   GROUP BY a.department_id) AS fc
ON dep.department_id = fc.department_id;
