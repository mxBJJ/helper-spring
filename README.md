# Model (Entity)

Uma boa maneira de começar um novo endpoint(ex: "/novo"), é criando a classe que irá ser responsável pelo modelo a ser retornado por ele.

## Exemplo de model:
```Java
@Entity
@Table(name = "tb_categories")
public class Category implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;
    private String name;

    @ManyToMany(mappedBy = "categories")
    private List<Product> products = new ArrayList<>();

    public Category() {
    }

    public Category(Integer id, String name) {
        this.id = id;
        this.name = name;
    }
    
   ```

Criando esta estrutura por exemplo, possuímos uma model de Categorias. O implements Serializable irá permitir que a classe seja serializada como JSON. O restante de código, é gerado pelos métodos getters e setters. A relação muitos para muitos não é utilizada no construtor, porque para criarmos uma categoria nova (instância do objeto), não precisamos declarar seus produtos de imediato.


# Repository (Repositório)

Ele irá tratar a camada de persistência dos dados e também será responsável por nos disponibilizar métodos "prontos" como findAll(), findById(), save(), delete(). No exemplo do petclinic, é criada uma classe de repositório ProductRepository mas ela "depende" de uma interface que extends um Repository, que recebe a Model que iremos trabalhar a persistência e o tipo do Id dessa model. O id pode ser Integer, Long...

## Exemplo de primeira interface:

```Java
public interface PetRepository {

    List<PetType> findAllPets() throws DataAccessException;
}

```


## Exemplo de segunda interface:

```Java

public interface SpringDataPetRepository extends PetRepository, Repository<Pet, Integer> {

}

```


# Service (Serviço)

O serviço terá a responsabilidade de conversar com o repositório.

##Exemplo de interface:

```Java

public interface ClinicService {

Collection<Pet> findAllPets() throws DataAccessException;
	void deletePet(Pet pet) throws DataAccessException;

}

```


## Exemplo de implementação dos métodos da interface:

```Java

@Service
public class ClinicServiceImpl implements ClinicService {

    private PetRepository petRepository;
    private VetRepository vetRepository;
    private OwnerRepository ownerRepository;
    private VisitRepository visitRepository;
    private SpecialtyRepository specialtyRepository;
	private PetTypeRepository petTypeRepository;

    @Autowired
     public ClinicServiceImpl(
       		 PetRepository petRepository) {
        this.petRepository = petRepository;
    }

	@Override
	@Transactional(readOnly = true)
	public Collection<Pet> findAllPets() throws DataAccessException {
		return petRepository.findAll();
	}

	@Override
	@Transactional
	public void deletePet(Pet pet) throws DataAccessException {
		petRepository.delete(pet);
	}
}

```

# Controller(Controlador)

Irá receber as requisições e acionar um serviço.


O método do controlador pode receber na requisição:

@RequestBody= Corpo da requisição enviado no formado JSON durante a requisição \

@RequestParam = São os parâmetros da requisição que chegarão no controlador através da url: "/exemplo/A=10&B=30". \

@PathVariable = é o parametro que nos diz o elemento a ser substituído por uma variavel na url, por exemplo: "users/{id}", esse {id} é enviado através de uma pathvariable, que é enviada na url da requisição, "users/1", esse 1 durante a requisição , chegará no méotodo do controlador que terá a rota users/{id}". \

Body e Param são bem visíveis ao utilizar o Postman.



## Exemplo de controlador

```Java

@RestController
@RequestMapping("api/pets")
public class PetRestController {

	@Autowired
	private ClinicService clinicService;

	@RequestMapping(value = "/{petId}", method = RequestMethod.GET)
	public ResponseEntity<Pet> getPet(@PathVariable("petId") int petId){
		Pet pet = this.clinicService.findPetById(petId);
    
		return new ResponseEntity<Pet>(pet, HttpStatus.OK);
	}
}

```


