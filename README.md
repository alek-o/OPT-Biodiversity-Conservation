
# 
---
# Problem background:
We are given a 2-dimensional grid $G$, which is formed by cells of various geometry. Each cell at the beginning can (doesn't have to) contain colony of each species out of $4$ species $A, M, E, O$. So one cell can have for example colonies $E$ and $O$, and another can have none.
Data of each grid cell:
```
- grid_id (column and row) or -grid_column and -grid_row
- dominant_land (land type, different species prefer different land types)
- cost_adaptation_A
- cost_adaptation_M
- cost_adaptation_E
- cost_adaptation_O
- cost_corridor
- has_A
- has_M
- has_E
- has_O
- geometry (not all are squares)
```
Species presence in each cell is defined by Boolean values `has_<species>`.
Each cell has a geometry parameter. Originally each cell can be a square, with estimated field of $\approx 0.5625km^2$. Due to the land variation and for example water present the polygon and ultimately field of land can be smaller than that, but not bigger. It is approximately 56.25 Hectares or 139 Acres.
# Problem goal
The goal is the conservation of biodiversity. 
To pursue the goal we have to:
- Expand their territories (more successful survival of each colony).
- Connect their territories (for each species individually) with corridors (prevents gene drift).
- Make sure territories do not connect by expanded land (kills variety).
- Minimize cost. (Can be done through constraints)
# Rules of territory expansion
- Each cell has a different expanding cost for each species.
- **CHANGE, MAKE IT EXPAND LIKE A CIRCLE:** Each newly expanded cell for species $s$ must be adjacent to at least one cell that is either an original colony cell of $s$ or an expanded cell of $s$. Cells connected horizontally or vertically are considered adjacent. Diagonal cells are not adjacent.
- Two territories must not get connected through territory expansion and form one bigger territory. It is forbidden, because it kills species variety.
- We must not expand the territory of species $M$ in a way, that it would intersect with both original and expanded territories of species $E$ and $O$, because $M$ is their natural predator. On the other hand $A$, $E$, $O$ can all intersect their territories.
- We should consider different geometry and normalize the price and conservation biodiversity accordingly to the area being expanded. Smaller area = less conservation and different cost/area ratio.
- It is not obligatory, to expand every territory, unless constraints definitely require it.
# Rules of corridors
- Corridor is a property of a cell.
- Can overlap with territory.
- Corridor cost can be different for each cell.
- Corridor path is formed by a chain of adjacent cells with corridors.
	- Example: cells {1,1}, {1,2}, {2,2} all having a corridor create a path connecting cells {1,1} and {2,2}.
- Corridors should form paths that connect separated territories of the same species throughout the grid.
	- Example: corridors should connect one territory of species $A$ with another territory of species $A$.
- Any colony with it's territory connected to a corridor can use that corridor. We must prevent situations, where species that exclude one another ($M$ exclude $O$,$E$ and vice versa) can trave through the same corridor.
- If a cell is adjacent to a territory and has a corridor, the corridor is connected to the territory.
- Corridor connects to a territory, when is has cell within the territory. 
	- Example: cells {1,1}, {1,3} are separate colonies, cells {1,1}, {1,2}, {1,3} are corridors. In this scenario, they connect both colonies. If we were to have only {1,1} and {1,2} as corridors, colonies would stay separated.
- Corridor can be going through a territory, but if not necessary, it should not, since it wastes money.	
- One corridor can accommodate multiple species, so it is not necessary to create two different corridors for e.g. species $A$ and species $O$, if their territories are close to one another.
- It is not obligatory, to connect every colony of species $s$ with any other colony of their species.
## Additional, general
- If there is any overlap between conflicting colonies, we ignore it and expand in a way, that would not worsen the situation. That is, we must not intersect more conflicting territories with our expansions.

## Edge cases for clarification
- If species has only one territory, there is no corridors for them required.
- If species appears in zero cells originally it cannot expand and there is no corridors for them required.
- If two original territories are connected, they are a single colony and considered the same territory.
- Expansion that would merge territories is strictly forbidden.
- Geometries are not adjacent if they share only a vertex.
- Corridors have to connect two territories directly even if going through different species territory. There is no indirect connections through different species territories.

# Linear programming part
## Goal
- Maximize conservation of biodiversity
- Minimize cost (can be handled as a fixed constraint)
## Constraints
### Territory size
#### Data
- Martes Martes
	- min: 1km2
	- mean: 10km2
	- max: 30km2
- Atelerix algirus
	- min: 0.3km2
	- mean: 3km2
	- max: 6km2
- Eliomys quercinus 
	- min: 0.5km2
	- mean: 2km2
	- max: 5km2
- Oryctolagus cuniculus 
	- min:0.2km2  
	- mean: 1.5km2, 
	- max: 3km2
#### The constraint
- The model HAS to create minimum space for each species.
- The model is going to encourage mean space for each species. (for example coefficient = 1)
- The model is going to encourage, but less for bigger space. (for example coefficient = 0.8)
- The model is going to discourage for much bigger space. (for example coefficient = 0.2)
### Territory availability
- land that is too dangerous and which type is completely unsuitable for certain species is NOT going to be allowed for expansion
- land of very good / good enough land is going to be distinguished by different coefficients.
# Additional Data Sources
## Martes Martes
### Territory size
https://animaldiversity.org/accounts/Martes_martes/
https://www.woodlandtrust.org.uk/trees-woods-and-wildlife/animals/mammals/pine-marten/
https://www.jstor.org/stable/3683493
## Atelerix alguris
### Territory size
https://www.wildlifeonline.me.uk/animals/article/european-hedgehog-territory-home-range

## Eliomys quercinus
### Territory size
https://link.springer.com/article/10.1007/s42991-021-00118-1
maybe the following??? but not really
https://frontiersinzoology.biomedcentral.com/articles/10.1186/s12983-017-0206-0
## Oryctolagus cuniculus
### Territory size
https://animaldiversity.org/accounts/Oryctolagus_cuniculus/
https://www.wildlifeonline.me.uk/animals/species/european-rabbit
https://pmc.ncbi.nlm.nih.gov/articles/PMC7158370/

### interesting
in terms of the distance of colonies
https://www.nature.com/articles/6885110
# Q&A
---
- Intersections of corridors, lands - cost is up to us, whether to spend $X\cdot\text{Species passing}$ depending on the number of species passing through the field or just $X$. $X$ being the cost of the corridor in the specified field.
- Predator $\text{Martes Martes}$ should not be intersecting their corridors or lands with potential pray, which are $\text{Dormice}$ and $\text{Rabbits}$.
