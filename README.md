
# OPT Project 1, UPV
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
No matter the geometry, the cells are still divided into squares. Adjacent still are the squares being in horizontal or vertical proximity.
# Problem goal
The goal is the conservation of biodiversity. 
To pursue the goal we have to:
- Expand their territories (more successful survival of each colony).
- Connect their territories (for each species individually) with corridors (prevents gene drift).
- Make sure territories do not connect by expanded land (kills variety).
- Minimize cost. (Can be done through constraints)

Additional notes about naming/conventions:
- A territory / colony (patches) we will call a single patch of certain species, which cells are all adjacent to other cells of the territory.
- Territories of the same species can connect via corridors and merge if their cells become adjacent. 
	- connections are desired
	- merges are forbidden
# Rules of territory expansion
- Each cell has a different expanding cost for each species.
- Each newly expanded cell for species $s$ must be adjacent to at least one cell that is either an original colony cell of $s$ or an expanded cell of $s$. Cells connected horizontally or vertically are considered adjacent. Diagonal cells are not adjacent.
- Each territorial patch should be elliptical or circular, expanding both vertically and horizontally proportionally. So they do not create straight lines or shapes that would not make sense in nature. Patches should follow elliptical shapes. Obviously as much as the grid allows. Technical explanation is going to be addressed in mathematical model.
- Two territories must not get connected through territory expansion and form one bigger territory. It is forbidden, because it kills species variety.
- We must not expand the territory of species $M$ in a way, that it would intersect (occupy the same cell) with both original and expanded territories of species $E$ or $O$, because $M$ is their natural predator. On the other hand $A$, $E$, $O$ can all intersect their territories.
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
- Any colony with it's territory connected to a corridor can use that corridor. So a corridor that has one of it's cells overlapping with a certain colony territory is accessible for it. We must prevent situations, where species that exclude one another ($M$ exclude $O$,$E$ and vice versa) can trave through the same corridor. So one corridor cannot have cells overlapping with cells having $M$ and $O$ or $M$ and $E$ at the same time.
- Corridor connects to a territory, when is has cell within the territory. 
	- Example: cells {1,1}, {1,3} are separate colonies, cells {1,1}, {1,2}, {1,3} are corridors. In this scenario, they connect both colonies. If we were to have only {1,1} and {1,2} as corridors, colonies would stay separated.
- Corridor can be going through a territory, but if not necessary, it should not, since it wastes money.	
- One corridor can accommodate multiple species, so it is not necessary to create two different corridors for e.g. species $A$ and species $O$, if their territories are close to one another.
- It is not obligatory, to connect every colony of species $s$ with any other colony of their species.
- Corridors may connect distinct territories but do not change territory identity. They are independent graph layers.
- Corridors do not merge territories, they only make connections between them. If we were to compare it to a graph, each territory still remains the same node, only new vertex appears.
## Additional, general
- If there is any overlap between initial conflicting colonies at the beginning, we ignore it and expand in a way, that would not worsen the situation. That is, we must not intersect more conflicting territories with our expansions.

## Edge cases for clarification
- If species has only one territory, there is no corridors for them required.
- If species appears in zero cells originally it cannot expand and there is no corridors for them required.
- If two original territories are connected, they are a single colony and considered the same territory.
- Expansion that would merge territories is strictly forbidden.

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

# sialala
```
x_s - current number of territory of species s
y_s - current number of corridors connecting disting colonies of species s
n_s – number of colonies of species s
m_s – number of REQUIRED cells (minimum) in the colonies of species s
l_s – avg for species s territory (in cells)
k – maximum number of cells for colony of species s
- n_s-1 - ( n-1 ) – maximum number of meaningful connections between colonies of species s (meaningful => complying with tree-pattern - done through constraints)
n_s*l_s-m_s - ( n*l - m ) – satisfactory number of cells for species s for acclimatization (we deal with one colony exceeding their mean size with constraints)
n_s*k_s-m_s – ( n*k - m ) - maximum number of cells per species s (once again we deal with it via constraints)

proportion of ( corridors / territory ):
(y/(n-1))/(x/(n*l-m)) = (y*(n*l-m))/(x*(n-1))
```