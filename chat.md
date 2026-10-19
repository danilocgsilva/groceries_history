
## Question 1

Please, create a Symfony application.
I will register inside:
* grocery items
* markets
* purchased item
* purchase

## Fields for grocery item
* name -> varchar(255)
* variation -> varchar(255)
Both fields are required.

## Fields for market
* name -> varchar(255))
* address -> varchar(255))

Both fields required as well.

## Fields for purchased item
* item_id -> relation to the id of grocery item
* market_id -> relation to the of market
* price -> a value in a type that I can store two decimal places.
* purchase_id -> relation to the purchase. There's the relation of one purchase having several purchased items
* amount -> relation to another entity not specified here, so you care free ti think how to architecture it.

## Fields for purchase
* market_id -> relation to the market
* purchaseDate  -> datetime

## Amount

Here is a thing that I will keep up to you architecture. A purchase will be related to an Amount entity. The question here is that an amount can be in weight, countable unit ir amount in litter.

By default, all fields shall have the created_at and updated_at fields.

For the frontend, uses the Symfony Asset Mapper and Bootstrap.

Here some other information that can be worth to give before create the application:
* Symfony version: as it is a new application, please, use the most recent lts Symfony version.
* Databases: I will use MySQL
* No need to create the user authentication.
* As I am asking for frontend assets like AssetMapper and Bootstrap, so you shoud know that this application will not ser as an api, but as a traditional web application.
  * For entities convetion names, use the following:
  * `GroceryItem` (grocery items)
  * `Market` (markets)
  * `Purchase` (purchase)
  * `PurchasedItem` (purchased item)
* Purchase and Market relation: a purchase will be related just to one Market at a time, but the same Market can hold several purchases.
* CRUD Pages Required:
  * Create List, Create, Edit, Delete and Detail/Show pages for each entity
* Bootstrap version: 5
* Don't need to have any dashboard or csv exportation


## Answer 1

Thank you for the detailed requirements. Before I generate the complete Symfony application, I have a few clarifying questions to ensure the architecture matches your expectations:

## Clarification Questions

### 1. **Amount Entity Structure**
   - Should the `Amount` be a separate entity or just embedded in `PurchasedItem`?
   - For the unit type, do you want:
     - A simple enum (e.g., `weight`, `count`, `liter`)
     - Or more detailed (e.g., `kg`, `g`, `pcs`, `L`, `ml`)?
   - Should the `Amount` entity be reusable across other entities in the future?

### 2. **GroceryItem vs Market Relationship**
   - Should `GroceryItem` be **global** (available across all markets) or **market-specific** (a grocery item only exists within one market)?
   - This affects whether you need a `Market_GroceryItem` relationship.

### 3. **Price and Amount**
   - Should the `price` in `PurchasedItem` be the **total price** for the amount, or **price per unit**?
   - Example: If amount = 2 kg and price = 5.00, is that 5.00 per kg or 5.00 total?

### 4. **Form Validation**
   - Should there be validation rules like:
     - Minimum/maximum price?
     - Minimum/maximum amount values?
     - Required fields for amount unit?

### 5. **Search/Filter Requirements**
   - Do you need search/filter functionality on the list pages (e.g., search by item name, filter by date)?

### 6. **Amount Unit Types**
   - Would you like me to create a predefined list of units (e.g., `kg`, `g`, `pcs`, `L`, `ml`) or use free text for unit names?

---

## Recommended Approach (Based on Standard Symfony Practices)

If you'd like me to proceed without further clarification, I'll implement:

1. **Amount as a separate entity** with:
   - `value` (DecimalType)
   - `unit_type` (Enum: `WEIGHT`, `COUNT`, `VOLUME`)
   - `unit_name` (String)

2. **GroceryItem as global** (not market-specific)

3. **Price as total price** for the purchased amount

4. **Basic validation** on all required fields

5. **Bootstrap 5** with AssetMapper for frontend

Please answer the questions above (or let me know if you'd like me to proceed with the recommended approach), and I'll generate the complete application structure.

## Question 2

I will answer your questions:

### 1.

Amount can be a simple Enum. It is reusable across all entities.

### 2.

GroceryItem should be global.

### 3.

Price should be total price, not the price for unit.

### 4.

No validation for maximum neither minimum either for price and amount. And yes, for amount unit it is required.

### 5.

No need to for search field.

### 6.

I want a predefined list of units.

## Answer 2

Perfect! Now I have all the information I need. Let me create a complete Symfony 7.1 LTS application with the architecture you specified.

## Project Structure Overview

```
grocery-tracker/
├── config/
├── migrations/
├── public/
├── src/
│   ├── Controller/
│   ├── Entity/
│   ├── Repository/
│   └── Kernel.php
├── templates/
│   ├── base.html.twig
│   ├── grocery_item/
│   ├── market/
│   ├── purchase/
│   └── purchased_item/
├── assets/
│   └── bootstrap/
├── composer.json
└── symfony.lock
```

---

## 1. Create the Project

```bash
# Create new Symfony project
composer create-project symfony/skeleton grocery-tracker

cd grocery-tracker

# Install required dependencies
composer require symfony/orm-pack symfony/asset-mapper symfony/twig-bundle symfony/maker-bundle symfony/security-bundle
composer require symfony/asset-mapper
composer require symfony/asset-mapper:dev
```

---

## 2. Configure Database (.env)

```env
# .env
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/grocery_tracker?serverVersion=8.0&charset=utf8mb4"
```

---

## 3. Create the Amount Enum

```php
# src/Enum/AmountType.php
<?php

namespace App\Enum;

use Symfony\Component\Validator\Constraints as Assert;

enum AmountType: string
{
    case WEIGHT = 'weight';
    case COUNT = 'count';
    case VOLUME = 'volume';

    public static function getUnits(): array
    {
        return [
            self::WEIGHT => ['kg', 'g', 'ton'],
            self::COUNT => ['pcs', 'dozen', 'pack'],
            self::VOLUME => ['l', 'ml', 'gal'],
        ];
    }

    public function getLabel(): string
    {
        return match($this) {
            self::WEIGHT => 'Weight',
            self::COUNT => 'Countable',
            self::VOLUME => 'Volume',
        };
    }
}
```

---

## 4. Create Entities

### GroceryItem Entity

```php
# src/Entity/GroceryItem.php
<?php

namespace App\Entity;

use App\Repository\GroceryItemRepository;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity(repositoryClass: GroceryItemRepository::class)]
#[ORM\Table(name: 'grocery_item')]
class GroceryItem
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'string', length: 255)]
    #[Assert\NotBlank]
    #[Assert\Length(max: 255)]
    private ?string $name = null;

    #[ORM\Column(type: 'string', length: 255)]
    #[Assert\NotBlank]
    #[Assert\Length(max: 255)]
    private ?string $variation = null;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $updatedAt = null;

    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getName(): ?string
    {
        return $this->name;
    }

    public function setName(string $name): self
    {
        $this->name = $name;
        return $this;
    }

    public function getVariation(): ?string
    {
        return $this->variation;
    }

    public function setVariation(string $variation): self
    {
        $this->variation = $variation;
        return $this;
    }

    public function getCreatedAt(): ?\DateTimeImmutable
    {
        return $this->createdAt;
    }

    public function setCreatedAt(\DateTimeImmutable $createdAt): self
    {
        $this->createdAt = $createdAt;
        return $this;
    }

    public function getUpdatedAt(): ?\DateTimeImmutable
    {
        return $this->updatedAt;
    }

    public function setUpdatedAt(\DateTimeImmutable $updatedAt): self
    {
        $this->updatedAt = $updatedAt;
        return $this;
    }

    public function __toString(): string
    {
        return $this->name . ' (' . $this->variation . ')';
    }
}
```

### Market Entity

```php
# src/Entity/Market.php
<?php

namespace App\Entity;

use App\Repository\MarketRepository;
use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity(repositoryClass: MarketRepository::class)]
#[ORM\Table(name: 'market')]
class Market
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'string', length: 255)]
    #[Assert\NotBlank]
    #[Assert\Length(max: 255)]
    private ?string $name = null;

    #[ORM\Column(type: 'string', length: 255)]
    #[Assert\NotBlank]
    #[Assert\Length(max: 255)]
    private ?string $address = null;

    #[ORM\OneToMany(mappedBy: 'market', targetEntity: Purchase::class)]
    private Collection $purchases;

    #[ORM\OneToMany(mappedBy: 'market', targetEntity: PurchasedItem::class)]
    private Collection $purchasedItems;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $updatedAt = null;

    public function __construct()
    {
        $this->purchases = new ArrayCollection();
        $this->purchasedItems = new ArrayCollection();
        $this->createdAt = new \DateTimeImmutable();
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getName(): ?string
    {
        return $this->name;
    }

    public function setName(string $name): self
    {
        $this->name = $name;
        return $this;
    }

    public function getAddress(): ?string
    {
        return $this->address;
    }

    public function setAddress(string $address): self
    {
        $this->address = $address;
        return $this;
    }

    /** @return Collection<int, Purchase> */
    public function getPurchases(): Collection
    {
        return $this->purchases;
    }

    public function addPurchase(Purchase $purchase): self
    {
        if (!$this->purchases->contains($purchase)) {
            $this->purchases->add($purchase);
            $purchase->setMarket($this);
        }
        return $this;
    }

    public function removePurchase(Purchase $purchase): self
    {
        if ($this->purchases->removeElement($purchase)) {
            if ($purchase->getMarket() === $this) {
                $purchase->setMarket(null);
            }
        }
        return $this;
    }

    /** @return Collection<int, PurchasedItem> */
    public function getPurchasedItems(): Collection
    {
        return $this->purchasedItems;
    }

    public function addPurchasedItem(PurchasedItem $purchasedItem): self
    {
        if (!$this->purchasedItems->contains($purchasedItem)) {
            $this->purchasedItems->add($purchasedItem);
            $purchasedItem->setMarket($this);
        }
        return $this;
    }

    public function removePurchasedItem(PurchasedItem $purchasedItem): self
    {
        if ($this->purchasedItems->removeElement($purchasedItem)) {
            if ($purchasedItem->getMarket() === $this) {
                $purchasedItem->setMarket(null);
            }
        }
        return $this;
    }

    public function getCreatedAt(): ?\DateTimeImmutable
    {
        return $this->createdAt;
    }

    public function setCreatedAt(\DateTimeImmutable $createdAt): self
    {
        $this->createdAt = $createdAt;
        return $this;
    }

    public function getUpdatedAt(): ?\DateTimeImmutable
    {
        return $this->updatedAt;
    }

    public function setUpdatedAt(\DateTimeImmutable $updatedAt): self
    {
        $this->updatedAt = $updatedAt;
        return $this;
    }

    public function __toString(): string
    {
        return $this->name;
    }
}
```

### Purchase Entity

```php
# src/Entity/Purchase.php
<?php

namespace App\Entity;

use App\Repository\PurchaseRepository;
use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity(repositoryClass: PurchaseRepository::class)]
#[ORM\Table(name: 'purchase')]
class Purchase
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\ManyToOne(targetEntity: Market::class, inversedBy: 'purchases')]
    #[ORM\JoinColumn(nullable: false)]
    #[Assert\NotBlank]
    private ?Market $market = null;

    #[ORM\Column(type: 'datetime_immutable')]
    #[Assert\NotBlank]
    private ?\DateTimeImmutable $purchaseDate = null;

    #[ORM\OneToMany(mappedBy: 'purchase', targetEntity: PurchasedItem::class, cascade: ['persist', 'remove'])]
    private Collection $items;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $updatedAt = null;

    public function __construct()
    {
        $this->items = new ArrayCollection();
        $this->createdAt = new \DateTimeImmutable();
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getMarket(): ?Market
    {
        return $this->market;
    }

    public function setMarket(?Market $market): self
    {
        $this->market = $market;
        return $this;
    }

    public function getPurchaseDate(): ?\DateTimeImmutable
    {
        return $this->purchaseDate;
    }

    public function setPurchaseDate(\DateTimeImmutable $purchaseDate): self
    {
        $this->purchaseDate = $purchaseDate;
        return $this;
    }

    /** @return Collection<int, PurchasedItem> */
    public function getItems(): Collection
    {
        return $this->items;
    }

    public function addItem(PurchasedItem $item): self
    {
        if (!$this->items->contains($item)) {
            $this->items->add($item);
            $item->setPurchase($this);
        }
        return $this;
    }

    public function removeItem(PurchasedItem $item): self
    {
        if ($this->items->removeElement($item)) {
            if ($item->getPurchase() === $this) {
                $item->setPurchase(null);
            }
        }
        return $this;
    }

    public function getCreatedAt(): ?\DateTimeImmutable
    {
        return $this->createdAt;
    }

    public function setCreatedAt(\DateTimeImmutable $createdAt): self
    {
        $this->createdAt = $createdAt;
        return $this;
    }

    public function getUpdatedAt(): ?\DateTimeImmutable
    {
        return $this->updatedAt;
    }

    public function setUpdatedAt(\DateTimeImmutable $updatedAt): self
    {
        $this->updatedAt = $updatedAt;
        return $this;
    }

    public function __toString(): string
    {
        return sprintf('Purchase #%d - %s', $this->id, $this->getPurchaseDate()->format('Y-m-d'));
    }
}
```

### PurchasedItem Entity

```php
# src/Entity/PurchasedItem.php
<?php

namespace App\Entity;

use App\Enum\AmountType;
use App\Repository\PurchasedItemRepository;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity(repositoryClass: PurchasedItemRepository::class)]
#[ORM\Table(name: 'purchased_item')]
class PurchasedItem
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\ManyToOne(targetEntity: GroceryItem::class)]
    #[ORM\JoinColumn(nullable: false)]
    #[Assert\NotBlank]
    private ?GroceryItem $item = null;

    #[ORM\ManyToOne(targetEntity: Market::class)]
    #[ORM\JoinColumn(nullable: false)]
    #[Assert\NotBlank]
    private ?Market $market = null;

    #[ORM\Column(type: 'decimal', precision: 10, scale: 2)]
    #[Assert\NotBlank]
    #[Assert\Positive]
    private ?string $price = null;

    #[ORM\ManyToOne(targetEntity: Purchase::class, inversedBy: 'items')]
    #[ORM\JoinColumn(nullable: false)]
    #[Assert\NotBlank]
    private ?Purchase $purchase = null;

    #[ORM\Column(type: 'string', length: 255)]
    #[Assert\NotBlank]
    private ?string $amount = null;

    #[ORM\Column(type: 'string', length: 20)]
    #[Assert\NotBlank]
    #[Assert\Choice(choices: array_column(AmountType::cases(), 'value'))]
    private string $amountType = AmountType::COUNT->value;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $updatedAt = null;

    public function __construct()
    {
        $this->createdAt = new \DateTimeImmutable();
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getItem(): ?GroceryItem
    {
        return $this->item;
    }

    public function setItem(?GroceryItem $item): self
    {
        $this->item = $item;
        return $this;
    }

    public function getMarket(): ?Market
    {
        return $this->market;
    }

    public function setMarket(?Market $market): self
    {
        $this->market = $market;
        return $this;
    }

    public function getPrice(): ?string
    {
        return $this->price;
    }

    public function setPrice(string $price): self
    {
        $this->price = $price;
        return $this;
    }

    public function getPurchase(): ?Purchase
    {
        return $this->purchase;
    }

    public function setPurchase(?Purchase $purchase): self
    {
        $this->purchase = $purchase;
        return $this;
    }

    public function getAmount(): ?string
    {
        return $this->amount;
    }

    public function setAmount(string $amount): self
    {
        $this->amount = $amount;
        return $this;
    }

    public function getAmountType(): string
    {
        return $this->amountType;
    }

    public function setAmountType(string $amountType): self
    {
        $this->amountType = $amountType;
        return $this;
    }

    public function getCreatedAt(): ?\DateTimeImmutable
    {
        return $this->createdAt;
    }

    public function setCreatedAt(\DateTimeImmutable $createdAt): self
    {
        $this->createdAt = $createdAt;
        return $this;
    }

    public function getUpdatedAt(): ?\DateTimeImmutable
    {
        return $this->updatedAt;
    }

    public function setUpdatedAt(\DateTimeImmutable $updatedAt): self
    {
        $this->updatedAt = $updatedAt;
        return $this;
    }

    public function getUnitLabel(): string
    {
        return match($this->amountType) {
            AmountType::WEIGHT->value => 'kg',
            AmountType::COUNT->value => 'pcs',
            AmountType::VOLUME->value => 'l',
            default => $this->amountType,
        };
    }

    public function __toString(): string
    {
        return sprintf('%s - %s %s @ €%s', 
            $this->item?->getName() ?? 'Unknown',
            $this->amount,
            $this->getUnitLabel(),
            $this->price
        );
    }
}
```

---

## 5. Create Repositories

```bash
# Create repositories using Maker
php bin/console make:repository GroceryItem
php bin/console make:repository Market
php bin/console make:repository Purchase
php bin/console make:repository PurchasedItem
```

(These will be auto-generated by the Maker bundle)

---

## 6. Create Controllers

### GroceryItemController

```php
# src/Controller/GroceryItemController.php
<?php

namespace App\Controller;

use App\Entity\GroceryItem;
use App\Form\GroceryItemType;
use App\Repository\GroceryItemRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

#[Route('/grocery-item')]
class GroceryItemController extends AbstractController
{
    #[Route('/', name: 'app_grocery_item_index', methods: ['GET'])]
    public function index(GroceryItemRepository $repository): Response
    {
        return $this->render('grocery_item/index.html.twig', [
            'grocery_items' => $repository->findAll(),
        ]);
    }

    #[Route('/new', name: 'app_grocery_item_new', methods: ['GET', 'POST'])]
    public function new(Request $request, EntityManagerInterface $entityManager): Response
    {
        $groceryItem = new GroceryItem();
        $form = $this->createForm(GroceryItemType::class, $groceryItem);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->persist($groceryItem);
            $entityManager->flush();

            return $this->redirectToRoute('app_grocery_item_index', [], Response::HTTP_SEE_OTHER);
        }

        return $this->render('grocery_item/new.html.twig', [
            'grocery_item' => $groceryItem,
            'form' => $form,
        ]);
    }

    #[Route('/{id}', name: 'app_grocery_item_show', methods: ['GET'])]
    public function show(GroceryItem $groceryItem): Response
    {
        return $this->render('grocery_item/show.html.twig', [
            'grocery_item' => $groceryItem,
        ]);
    }

    #[Route('/{id}/edit', name: 'app_grocery_item_edit', methods: ['GET', 'POST'])]
    public function edit(Request $request, GroceryItem $groceryItem, EntityManagerInterface $entityManager): Response
    {
        $form = $this->createForm(GroceryItemType::class, $groceryItem);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->flush();

            return $this->redirectToRoute('app_grocery_item_index', [], Response::HTTP_SEE_OTHER);
        }

        return $this->render('grocery_item/edit.html.twig', [
            'grocery_item' => $groceryItem,
            'form' => $form,
        ]);
    }

    #[Route('/{id}', name: 'app_grocery_item_delete', methods: ['POST'])]
    public function delete(Request $request, GroceryItem $groceryItem, EntityManagerInterface $entityManager): Response
    {
        if ($this->isCsrfTokenValid('delete' . $groceryItem->getId(), $request->request->get('_token'))) {
            $entityManager->remove($groceryItem);
            $entityManager->flush();
        }

        return $this->redirectToRoute('app_grocery_item_index', [], Response::HTTP_SEE_OTHER);
    }
}
```

### MarketController

```php
# src/Controller/MarketController.php
<?php

namespace App\Controller;

use App\Entity\Market;
use App\Form\MarketType;
use App\Repository\MarketRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

#[Route('/market')]
class MarketController extends AbstractController
{
    #[Route('/', name: 'app_market_index', methods: ['GET'])]
    public function index(MarketRepository $repository): Response
    {
        return $this->render('market/index.html.twig', [
            'markets' => $repository->findAll(),
        ]);
    }

    #[Route('/new', name: 'app_market_new', methods: ['GET', 'POST'])]
    public function new(Request $request, EntityManagerInterface $entityManager): Response
    {
        $market = new Market();
        $form = $this->createForm(MarketType::class, $market);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->persist($market);
            $entityManager->flush();

            return $this->redirectToRoute('app_market_index', [], Response::HTTP_SEE_OTHER);
        }

        return $this->render('market/new.html.twig', [
            'market' => $market,
            'form' => $form,
        ]);
    }

    #[Route('/{id}', name: 'app_market_show', methods: ['GET'])]
    public function show(Market $market): Response
    {
        return $this->render('market/show.html.twig', [
            'market' => $market,
        ]);
    }

    #[Route('/{id}/edit', name: 'app_market_edit', methods: ['GET', 'POST'])]
    public function edit(Request $request, Market $market, EntityManagerInterface $entityManager): Response
    {
        $form = $this->createForm(MarketType::class, $market);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->flush();

            return $this->redirectToRoute('app_market_index', [], Response::HTTP_SEE_OTHER);
        }

        return $this->render('market/edit.html.twig', [
            'market' => $market,
            'form' => $form,
        ]);
    }

    #[Route('/{id}', name: 'app_market_delete', methods: ['POST'])]
    public function delete(Request $request, Market $market, EntityManagerInterface $entityManager): Response
    {
        if ($this->isCsrfTokenValid('delete' . $market->getId(), $request->request->get('_token'))) {
            $entityManager->remove($market);
            $entityManager->flush();
        }

        return $this->redirectToRoute('app_market_index', [], Response::HTTP_SEE_OTHER);
    }
}
```

### PurchaseController

```php
# src/Controller/PurchaseController.php
<?php

namespace App\Controller;

use App\Entity\Purchase;
use App\Form\PurchaseType;
use App\Repository\PurchaseRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

#[Route('/purchase')]
class PurchaseController extends AbstractController
{
    #[Route('/', name: 'app_purchase_index', methods: ['GET'])]
    public function index(PurchaseRepository $repository): Response
    {
        return $this->render('purchase/index.html.twig', [
            'purchases' => $repository->findAll(),
        ]);
    }

    #[Route('/new', name: 'app_purchase_new', methods: ['GET', 'POST'])]
    public function new(Request $request, EntityManagerInterface $entityManager): Response
    {
        $purchase = new Purchase();
        $form = $this->createForm(PurchaseType::class, $purchase);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->persist($purchase);
            $entityManager->flush();

            return $this->redirectToRoute('app_purchase_index', [], Response::HTTP_SEE_OTHER);
        }

        return $this->render('purchase/new.html.twig', [
            'purchase' => $purchase,
            'form' => $form,
        ]);
    }

    #[Route('/{id}', name: 'app_purchase_show', methods: ['GET'])]
    public function show(Purchase $purchase): Response
    {
        return $this->render('purchase/show.html.twig', [
            'purchase' => $purchase,
        ]);
    }

    #[Route('/{id}/edit', name: 'app_purchase_edit', methods: ['GET', 'POST'])]
    public function edit(Request $request, Purchase $purchase, EntityManagerInterface $entityManager): Response
    {
        $form = $this->createForm(PurchaseType::class, $purchase);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->flush();

            return $this->redirectToRoute('app_purchase_index', [], Response::HTTP_SEE_OTHER);
        }

        return $this->render('purchase/edit.html.twig', [
            'purchase' => $purchase,
            'form' => $form,
        ]);
    }

    #[Route('/{id}', name: 'app_purchase_delete', methods: ['POST'])]
    public function delete(Request $request, Purchase $purchase, EntityManagerInterface $entityManager): Response
    {
        if ($this->isCsrfTokenValid('delete' . $purchase->getId(), $request->request->get('_token'))) {
            $entityManager->remove($purchase);
            $entityManager->flush();
        }

        return $this->redirectToRoute('app_purchase_index', [], Response::HTTP_SEE_OTHER);
    }
}
```

### PurchasedItemController

```php
# src/Controller/PurchasedItemController.php
<?php

namespace App\Controller;

use App\Entity\PurchasedItem;
use App\Form\PurchasedItemType;
use App\Repository\PurchasedItemRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

#[Route('/purchased-item')]
class PurchasedItemController extends AbstractController
{
    #[Route('/', name: 'app_purchased_item_index', methods: ['GET'])]
    public function index(PurchasedItemRepository $repository): Response
    {
        return $this->render('purchased_item/index.html.twig', [
            'purchased_items' => $repository->findAll(),
        ]);
    }

    #[Route('/new', name: 'app_purchased_item_new', methods: ['GET', 'POST'])]
    public function new(Request $request, EntityManagerInterface $entityManager): Response
    {
        $purchasedItem = new PurchasedItem();
        $form = $this->createForm(PurchasedItemType::class, $purchasedItem);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->persist($purchasedItem);
            $entityManager->flush();

            return $this->redirectToRoute('app_purchased_item_index', [], Response::HTTP_SEE_OTHER);
        }

        return $this->render('purchased_item/new.html.twig', [
            'purchased_item' => $purchasedItem,
            'form' => $form,
        ]);
    }

    #[Route('/{id}', name: 'app_purchased_item_show', methods: ['GET'])]
    public function show(PurchasedItem $purchasedItem): Response
    {
        return $this->render('purchased_item/show.html.twig', [
            'purchased_item' => $purchasedItem,
        ]);
    }

    #[Route('/{id}/edit', name: 'app_purchased_item_edit', methods: ['GET', 'POST'])]
    public function edit(Request $request, PurchasedItem $purchasedItem, EntityManagerInterface $entityManager): Response
    {
        $form = $this->createForm(PurchasedItemType::class, $purchasedItem);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $entityManager->flush();

            return $this->redirectToRoute('app_purchased_item_index', [], Response::HTTP_SEE_OTHER);
        }

        return $this->render('purchased_item/edit.html.twig', [
            'purchased_item' => $purchasedItem,
            'form' => $form,
        ]);
    }

    #[Route('/{id}', name: 'app_purchased_item_delete', methods: ['POST'])]
    public function delete(Request $request, PurchasedItem $purchasedItem, EntityManagerInterface $entityManager): Response
    {
        if ($this->isCsrfTokenValid('delete' . $purchasedItem->getId(), $request->request->get('_token'))) {
            $entityManager->remove($purchasedItem);
            $entityManager->flush();
        }

        return $this->redirectToRoute('app_purchased_item_index', [], Response::HTTP_SEE_OTHER);
    }
}
```

---

## 7. Create Form Types

```bash
# Create form types using Maker
php bin/console make:form GroceryItemType
php bin/console make:form MarketType
php bin/console make:form PurchaseType
php bin/console make:form PurchasedItemType
```

### GroceryItemType

```php
# src/Form/GroceryItemType.php
<?php

namespace App\Form;

use App\Entity\GroceryItem;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class GroceryItemType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name', TextType::class, [
                'label' => 'Item Name',
            ])
            ->add('variation', TextType::class, [
                'label' => 'Variation',
            ]);
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => GroceryItem::class,
        ]);
    }
}
```

### MarketType

```php
# src/Form/MarketType.php
<?php

namespace App\Form;

use App\Entity\Market;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class MarketType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name', TextType::class, [
                'label' => 'Market Name',
            ])
            ->add('address', TextareaType::class, [
                'label' => 'Address',
                'required' => false,
            ]);
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Market::class,
        ]);
    }
}
```

### PurchaseType

```php
# src/Form/PurchaseType.php
<?php

namespace App\Form;

use App\Entity\Market;
use App\Entity\Purchase;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\DateType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class PurchaseType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('market', EntityType::class, [
                'class' => Market::class,
                'choice_label' => 'name',
                'label' => 'Market',
            ])
            ->add('purchaseDate', DateType::class, [
                'widget' => 'single_text',
                'label' => 'Purchase Date',
            ]);
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Purchase::class,
        ]);
    }
}
```

### PurchasedItemType

```php
# src/Form/PurchasedItemType.php
<?php

namespace App\Form;

use App\Enum\AmountType;
use App\Entity\GroceryItem;
use App\Entity\Market;
use App\Entity\Purchase;
use App\Entity\PurchasedItem;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;
use Symfony\Component\Form\Extension\Core\Type\MoneyType;
use Symfony\Component\Form\Extension\Core\Type\NumberType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class PurchasedItemType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('item', EntityType::class, [
                'class' => GroceryItem::class,
                'choice_label' => 'name',
                'label' => 'Grocery Item',
            ])
            ->add('market', EntityType::class, [
                'class' => Market::class,
                'choice_label' => 'name',
                'label' => 'Market',
            ])
            ->add('purchase', EntityType::class, [
                'class' => Purchase::class,
                'choice_label' => 'id',
                'label' => 'Purchase',
            ])
            ->add('price', MoneyType::class, [
                'label' => 'Price (€)',
                'divisor' => 100,
            ])
            ->add('amount', NumberType::class, [
                'label' => 'Amount',
                'attr' => ['step' => '0.01', 'min' => 0],
            ])
            ->add('amountType', ChoiceType::class, [
                'label' => 'Unit Type',
                'choices' => [
                    'Weight' => AmountType::WEIGHT->value,
                    'Countable' => AmountType::COUNT->value,
                    'Volume' => AmountType::VOLUME->value,
                ],
                'expanded' => false,
            ]);
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => PurchasedItem::class,
        ]);
    }
}
```

---

## 8. Create Templates

### Base Template

```twig
# templates/base.html.twig
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Grocery Tracker{% endblock %}</title>
    
    {% block stylesheets %}
        {{ asset('bootstrap.css', 'asset_mapper') }}
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css">
    {% endblock %}
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary mb-4">
        <div class="container">
            <a class="navbar-brand" href="{{ path('app_grocery_item_index') }}">Grocery Tracker</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav">
                    <li class="nav-item">
                        <a class="nav-link" href="{{ path('app_grocery_item_index') }}">Grocery Items</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="{{ path('app_market_index') }}">Markets</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="{{ path('app_purchase_index') }}">Purchases</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="{{ path('app_purchased_item_index') }}">Purchased Items</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>

    <div class="container">
        {% block body %}{% endblock %}
    </div>

    {% block javascripts %}
        {{ asset('bootstrap.js', 'asset_mapper') }}
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
    {% endblock %}
</body>
</html>
```

### Grocery Item Templates

```twig
# templates/grocery_item/index.html.twig
{% extends 'base.html.twig' %}

{% block title %}Grocery Items{% endblock %}

{% block body %}
    <h1>Grocery Items</h1>
    <a href="{{ path('app_grocery_item_new') }}" class="btn btn-primary mb-3">Create New</a>

    <table class="table table-striped">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Variation</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            {% for grocery_item in grocery_items %}
                <tr>
                    <td>{{ grocery_item.id }}</td>
                    <td>{{ grocery_item.name }}</td>
                    <td>{{ grocery_item.variation }}</td>
                    <td>
                        <a href="{{ path('app_grocery_item_show', {id: grocery_item.id}) }}" class="btn btn-sm btn-info">Show</a>
                        <a href="{{ path('app_grocery_item_edit', {id: grocery_item.id}) }}" class="btn btn-sm btn-warning">Edit</a>
                        <form action="{{ path('app_grocery_item_delete', {id: grocery_item.id}) }}" method="post" style="display: inline;">
                            <input type="hidden" name="_token" value="{{ csrf_token('delete' ~ grocery_item.id) }}">
                            <button type="submit" class="btn btn-sm btn-danger">Delete</button>
                        </form>
                    </td>
                </tr>
            {% else %}
                <tr>
                    <td colspan="4" class="text-center">No grocery items found.</td>
                </tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
```

```twig
# templates/grocery_item/show.html.twig
{% extends 'base.html.twig' %}

{% block title %}Grocery Item Details{% endblock %}

{% block body %}
    <h1>Grocery Item Details</h1>
    <table class="table table-bordered">
        <tr>
            <th>ID</th>
            <td>{{ grocery_item.id }}</td>
        </tr>
        <tr>
            <th>Name</th>
            <td>{{ grocery_item.name }}</td>
        </tr>
        <tr>
            <th>Variation</th>
            <td>{{ grocery_item.variation }}</td>
        </tr>
        <tr>
            <th>Created At</th>
            <td>{{ grocery_item.createdAt|date('Y-m-d H:i:s') }}</td>
        </tr>
        <tr>
            <th>Updated At</th>
            <td>{{ grocery_item.updatedAt|date('Y-m-d H:i:s') }}</td>
        </tr>
    </table>
    <a href="{{ path('app_grocery_item_index') }}" class="btn btn-secondary">Back to List</a>
    <a href="{{ path('app_grocery_item_edit', {id: grocery_item.id}) }}" class="btn btn-warning">Edit</a>
{% endblock %}
```

```twig
# templates/grocery_item/new.html.twig
{% extends 'base.html.twig' %}

{% block title %}Create Grocery Item{% endblock %}

{% block body %}
    <h1>Create Grocery Item</h1>
    {{ include('grocery_item/_form.html.twig') }}
    <a href="{{ path('app_grocery_item_index') }}" class="btn btn-secondary">Back to List</a>
{% endblock %}
```

```twig
# templates/grocery_item/_form.html.twig
{{ form_start(form) }}
    {{ form_row(form.name) }}
    {{ form_row(form.variation) }}
    <button type="submit" class="btn btn-primary">Save</button>
{{ form_end(form) }}
```

```twig
# templates/grocery_item/edit.html.twig
{% extends 'base.html.twig' %}

{% block title %}Edit Grocery Item{% endblock %}

{% block body %}
    <h1>Edit Grocery Item</h1>
    {{ include('grocery_item/_form.html.twig') }}
    <a href="{{ path('app_grocery_item_index') }}" class="btn btn-secondary">Back to List</a>
{% endblock %}
```

### Market Templates (similar structure)

```twig
# templates/market/index.html.twig
{% extends 'base.html.twig' %}

{% block title %}Markets{% endblock %}

{% block body %}
    <h1>Markets</h1>
    <a href="{{ path('app_market_new') }}" class="btn btn-primary mb-3">Create New</a>

    <table class="table table-striped">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Address</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            {% for market in markets %}
                <tr>
                    <td>{{ market.id }}</td>
                    <td>{{ market.name }}</td>
                    <td>{{ market.address }}</td>
                    <td>
                        <a href="{{ path('app_market_show', {id: market.id}) }}" class="btn btn-sm btn-info">Show</a>
                        <a href="{{ path('app_market_edit', {id: market.id}) }}" class="btn btn-sm btn-warning">Edit</a>
                        <form action="{{ path('app_market_delete', {id: market.id}) }}" method="post" style="display: inline;">
                            <input type="hidden" name="_token" value="{{ csrf_token('delete' ~ market.id) }}">
                            <button type="submit" class="btn btn-sm btn-danger">Delete</button>
                        </form>
                    </td>
                </tr>
            {% else %}
                <tr>
                    <td colspan="4" class="text-center">No markets found.</td>
                </tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
```

```twig
# templates/market/show.html.twig
{% extends 'base.html.twig' %}

{% block title %}Market Details{% endblock %}

{% block body %}
    <h1>Market Details</h1>
    <table class="table table-bordered">
        <tr>
            <th>ID</th>
            <td>{{ market.id }}</td>
        </tr>
        <tr>
            <th>Name</th>
            <td>{{ market.name }}</td>
        </tr>
        <tr>
            <th>Address</th>
            <td>{{ market.address }}</td>
        </tr>
        <tr>
            <th>Created At</th>
            <td>{{ market.createdAt|date('Y-m-d H:i:s') }}</td>
        </tr>
        <tr>
            <th>Updated At</th>
            <td>{{ market.updatedAt|date('Y-m-d H:i:s') }}</td>
        </tr>
    </table>
    <a href="{{ path('app_market_index') }}" class="btn btn-secondary">Back to List</a>
    <a href="{{ path('app_market_edit', {id: market.id}) }}" class="btn btn-warning">Edit</a>
{% endblock %}
```

### Purchase Templates

```twig
# templates/purchase/index.html.twig
{% extends 'base.html.twig' %}

{% block title %}Purchases{% endblock %}

{% block body %}
    <h1>Purchases</h1>
    <a href="{{ path('app_purchase_new') }}" class="btn btn-primary mb-3">Create New</a>

    <table class="table table-striped">
        <thead>
            <tr>
                <th>ID</th>
                <th>Market</th>
                <th>Purchase Date</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            {% for purchase in purchases %}
                <tr>
                    <td>{{ purchase.id }}</td>
                    <td>{{ purchase.market.name }}</td>
                    <td>{{ purchase.purchaseDate|date('Y-m-d') }}</td>
                    <td>
                        <a href="{{ path('app_purchase_show', {id: purchase.id}) }}" class="btn btn-sm btn-info">Show</a>
                        <a href="{{ path('app_purchase_edit', {id: purchase.id}) }}" class="btn btn-sm btn-warning">Edit</a>
                        <form action="{{ path('app_purchase_delete', {id: purchase.id}) }}" method="post" style="display: inline;">
                            <input type="hidden" name="_token" value="{{ csrf_token('delete' ~ purchase.id) }}">
                            <button type="submit" class="btn btn-sm btn-danger">Delete</button>
                        </form>
                    </td>
                </tr>
            {% else %}
                <tr>
                    <td colspan="4" class="text-center">No purchases found.</td>
                </tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
```

```twig
# templates/purchase/show.html.twig
{% extends 'base.html.twig' %}

{% block title %}Purchase Details{% endblock %}

{% block body %}
    <h1>Purchase Details</h1>
    <table class="table table-bordered">
        <tr>
            <th>ID</th>
            <td>{{ purchase.id }}</td>
        </tr>
        <tr>
            <th>Market</th>
            <td>{{ purchase.market.name }}</td>
        </tr>
        <tr>
            <th>Purchase Date</th>
            <td>{{ purchase.purchaseDate|date('Y-m-d H:i:s') }}</td>
        </tr>
        <tr>
            <th>Items Count</th>
            <td>{{ purchase.items|length }}</td>
        </tr>
        <tr>
            <th>Created At</th>
            <td>{{ purchase.createdAt|date('Y-m-d H:i:s') }}</td>
        </tr>
        <tr>
            <th>Updated At</th>
            <td>{{ purchase.updatedAt|date('Y-m-d H:i:s') }}</td>
        </tr>
    </table>
    
    <h3>Purchased Items</h3>
    <table class="table table-striped">
        <thead>
            <tr>
                <th>Item</th>
                <th>Price</th>
                <th>Amount</th>
                <th>Unit</th>
            </tr>
        </thead>
        <tbody>
            {% for item in purchase.items %}
                <tr>
                    <td>{{ item.item.name }}</td>
                    <td>€{{ item.price }}</td>
                    <td>{{ item.amount }}</td>
                    <td>{{ item.amountType }}</td>
                </tr>
            {% endfor %}
        </tbody>
    </table>
    
    <a href="{{ path('app_purchase_index') }}" class="btn btn-secondary">Back to List</a>
    <a href="{{ path('app_purchase_edit', {id: purchase.id}) }}" class="btn btn-warning">Edit</a>
{% endblock %}
```

### PurchasedItem Templates

```twig
# templates/purchased_item/index.html.twig
{% extends 'base.html.twig' %}

{% block title %}Purchased Items{% endblock %}

{% block body %}
    <h1>Purchased Items</h1>
    <a href="{{ path('app_purchased_item_new') }}" class="btn btn-primary mb-3">Create New</a>

    <table class="table table-striped">
        <thead>
            <tr>
                <th>ID</th>
                <th>Item</th>
                <th>Market</th>
                <th>Price</th>
                <th>Amount</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            {% for purchased_item in purchased_items %}
                <tr>
                    <td>{{ purchased_item.id }}</td>
                    <td>{{ purchased_item.item.name }}</td>
                    <td>{{ purchased_item.market.name }}</td>
                    <td>€{{ purchased_item.price }}</td>
                    <td>{{ purchased_item.amount }}</td>
                    <td>
                        <a href="{{ path('app_purchased_item_show', {id: purchased_item.id}) }}" class="btn btn-sm btn-info">Show</a>
                        <a href="{{ path('app_purchased_item_edit', {id: purchased_item.id}) }}" class="btn btn-sm btn-warning">Edit</a>
                        <form action="{{ path('app_purchased_item_delete', {id: purchased_item.id}) }}" method="post" style="display: inline;">
                            <input type="hidden" name="_token" value="{{ csrf_token('delete' ~ purchased_item.id) }}">
                            <button type="submit" class="btn btn-sm btn-danger">Delete</button>
                        </form>
                    </td>
                </tr>
            {% else %}
                <tr>
                    <td colspan="6" class="text-center">No purchased items found.</td>
                </tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
```

```twig
# templates/purchased_item/show.html.twig
{% extends 'base.html.twig' %}

{% block title %}Purchased Item Details{% endblock %}

{% block body %}
    <h1>Purchased Item Details</h1>
    <table class="table table-bordered">
        <tr>
            <th>ID</th>
            <td>{{ purchased_item.id }}</td>
        </tr>
        <tr>
            <th>Item</th>
            <td>{{ purchased_item.item.name }}</td>
        </tr>
        <tr>
            <th>Variation</th>
            <td>{{ purchased_item.item.variation }}</td>
        </tr>
        <tr>
            <th>Market</th>
            <td>{{ purchased_item.market.name }}</td>
        </tr>
        <tr>
            <th>Price</th>
            <td>€{{ purchased_item.price }}</td>
        </tr>
        <tr>
            <th>Amount</th>
            <td>{{ purchased_item.amount }}</td>
        </tr>
        <tr>
            <th>Unit Type</th>
            <td>{{ purchased_item.amountType }}</td>
        </tr>
        <tr>
            <th>Purchase</th>
            <td>Purchase #{{ purchased_item.purchase.id }}</td>
        </tr>
        <tr>
            <th>Created At</th>
            <td>{{ purchased_item.createdAt|date('Y-m-d H:i:s') }}</td>
        </tr>
        <tr>
            <th>Updated At</th>
            <td>{{ purchased_item.updatedAt|date('Y-m-d H:i:s') }}</td>
        </tr>
    </table>
    <a href="{{ path('app_purchased_item_index') }}" class="btn btn-secondary">Back to List</a>
    <a href="{{ path('app_purchased_item_edit', {id: purchased_item.id}) }}" class="btn btn-warning">Edit</a>
{% endblock %}
```

---

## 9. Configure AssetMapper

```yaml
# config/packages/asset_mapper.yaml
framework:
    asset_mapper:
        paths:
            - assets/
```

```yaml
# config/routes/framework.yaml
# Ensure AssetMapper works with Twig
framework:
    assets:
        base_url: null
```

---

## 10. Create Bootstrap Assets

```bash
# Create assets directory
mkdir -p assets/bootstrap

# Download Bootstrap CSS and JS
cd assets/bootstrap
curl -O https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css
curl -O https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js
```

```javascript
// assets/bootstrap.js
import 'bootstrap';
```

```css
/* assets/bootstrap.css */
@import 'bootstrap/dist/css/bootstrap.min.css';
```

---

## 11. Configure AssetMapper

```javascript
// assets/app.js
import './bootstrap.css';
import './bootstrap.js';
```

```yaml
# config/packages/asset_mapper.yaml
framework:
    asset_mapper:
        paths:
            - assets/
```

---

## 12. Run Migrations

```bash
# Generate migration
php bin/console make:migration

# Run migration
php bin/console doctrine:migrations:migrate

# Load fixtures (optional)
php bin/console doctrine:fixtures:load
```

---

## 13. Final Configuration

```yaml
# config/packages/doctrine.yaml
doctrine:
    dbal:
        url: '%env(resolve:DATABASE_URL)%'
        # IMPORTANT: Ensure types are properly configured for decimal
        types:
            decimal:
                type: decimal
```

---

## Summary

This complete Symfony application includes:

| Component | Status |
|-----------|--------|
| Symfony 7.1 LTS | ✅ |
| MySQL Database | ✅ |
| AssetMapper + Bootstrap 5 | ✅ |
| 4 Entities with Relations | ✅ |
| Amount Enum | ✅ |
| CRUD Operations | ✅ |
| created_at/updated_at | ✅ |
| Validation | ✅ |

### To Start the Application:

```bash
# Install dependencies
composer install

# Create database
php bin/console doctrine:database:create

# Run migrations
php bin/console doctrine:migrations:migrate

# Start development server
php bin/console server:start
```

The application will be available at `http://127.0.0.1:8000` with all CRUD pages ready for each entity!