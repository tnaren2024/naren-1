ConsumerUsecasePage 
// pages/consumerPage/consumerUsecasePage.js
class ConsumerUsecasePage {
  constructor(page) {
    this.page = page;
    this.dashboardTab = page.locator('#dashboardTab');
    this.documentationTab = page.locator('#documentationTab');
    this.addNewUseCaseButton = page.locator('#addNewUseCase');
    this.useCaseNameInput = page.locator('#useCaseName');
    this.multiSelectInput = page.locator('#multiselect-4');
    this.totalAmountInput = page.locator('#totalAmt');
    this.projectDetailInput = page.locator('#projectDetail');
    this.addButton = page.locator('#addBtn');
    this.viewUseCaseButton = page.locator('(//button[normalize-space()="View Use Case"])[1]');
    this.accessControlNavItem = page.locator('#accessControlNavItem');
    this.descriptionInput = page.locator('#description');
    this.carIdInput = page.locator('#carId');
    this.validateButton = page.locator('#valBtn');
    this.nextButton = page.locator('button:has-text("Next")');
    this.locationDropdown = page.locator('#location');
    this.selectSourceButton = page.locator('(//button[@id="select_source_btn"])[1]');
  }

  // Method to generate a random name
  generateRandomName(prefix = 'UseCase') {
    const randomString = Math.random().toString(36).substring(2, 8); // Generates a random 6-character string
    return `${prefix}_${randomString}`;
  }

  async navigateToDashboardTab() {
    await this.dashboardTab.click();
  }

  async getDashboardTabText() {
    return await this.dashboardTab.textContent();
  }

  async getDocumentationTabText() {
    await this.documentationTab.click();
    return await this.documentationTab.textContent();
  }

  async addNewUseCase(name, amount, projectDetail) {
    await this.addNewUseCaseButton.click();
    await this.useCaseNameInput.fill(name);
    await this.totalAmountInput.fill(amount);
    await this.projectDetailInput.fill(projectDetail);
    await this.addButton.click();
  }

  async viewUseCaseDetails() {
    await this.viewUseCaseButton.click();
  }

  async fillUseCaseDetails(description, carId) {
    await this.descriptionInput.fill(description);
    await this.carIdInput.fill(carId);
    await this.validateButton.click();
  }

  async proceedToNextStep() {
    await this.nextButton.click();
  }

  async selectLocationAndSource(location) {
    await this.locationDropdown.selectOption(location);
    await this.selectSourceButton.click();
  }

  async checkElementVisibility(selector) {
    await expect(this.page.locator(selector)).toBeVisible();
  }
}

export default ConsumerUsecasePage;
//=============================================

// consumerUsecaseTest.spec.js
import { test, expect } from "@playwright/test";
import HomePage from "../../pages/commonPage/homePage.js";
import ConsumerUsecasePage from "../../pages/consumerPage/consumerUsecasePage.js";

test.describe("Consumer Dashboard validation", () => {
  let page;
  let homePage;
  let consumerUsecasePage;

  test.beforeEach(async ({ browser }) => {
    page = await browser.newPage();
    homePage = new HomePage(page);
    consumerUsecasePage = new ConsumerUsecasePage(page);

    await page.goto("https://hyperdrive-dev.aexp.com/");
    await homePage.login("babu", "Hyperdrive@UI@5814");
    await homePage.navigateToConsumerDashboard();
  });

  test("Verify Consumer Dashboard tab text", async () => {
    await consumerUsecasePage.navigateToDashboardTab();
    const actualText = await consumerUsecasePage.getDashboardTabText();
    expect.soft(actualText).toContain("Consumer Dashboard");
  });

  test("Verify Add New Use Case tab text", async () => {
    await consumerUsecasePage.navigateToDashboardTab();
    const addNewUseCaseText = await consumerUsecasePage.getDashboardTabText();
    expect.soft(addNewUseCaseText).toContain("Add New Use Case");
  });

  test("Verify expandAllText and collapseAllText text", async () => {
    await consumerUsecasePage.navigateToDashboardTab();
    const expandAllText = await consumerUsecasePage.getDashboardTabText();
    expect.soft(expandAllText).toContain("Expand All");

    const collapseAllText = await consumerUsecasePage.getDashboardTabText();
    expect.soft(collapseAllText).toContain("Collapse");
  });

  test("Verify DOCUMENTATION tab text", async () => {
    const docTabText = await consumerUsecasePage.getDocumentationTabText();
    expect.soft(docTabText).toContain("DOCUMENTATION");
  });

  test("Add New Use Case", async () => {
    const randomName = consumerUsecasePage.generateRandomName(); // Generate a unique name
    await consumerUsecasePage.addNewUseCase(randomName, '5555', 'https://hyperdrive-dev.aexp.com/');
    await expect(page.locator('#addSuccessMessage')).toBeVisible();
  });

  test("Verify Additional Headers and Texts", async () => {
    await consumerUsecasePage.addNewUseCase('Sample', '500', 'https://example.com');
    await consumerUsecasePage.viewUseCaseDetails();

    await expect(page.locator('h1:has-text("Add Use Case")')).toBeVisible();
    await expect(page.locator('h1:has-text("Identification")')).toBeVisible();
    await expect(page.locator('label:has-text("Impact Categories")')).toBeVisible();
  });

  test("Negative Case - Invalid Project Detail Link", async () => {
    await consumerUsecasePage.addNewUseCase('InvalidCase', '1000', 'https://mail.google.com/');
    const errorMsg = await page.locator('#errorMessage').textContent();
    expect.soft(errorMsg).toContain("Please enter a correct link with ending .aexp.com");
  });

  test("Define Use Case Purpose and proceed through flow", async () => {
    await consumerUsecasePage.viewUseCaseDetails();
    await consumerUsecasePage.fillUseCaseDetails('Description Sample', '500000138');
    await consumerUsecasePage.proceedToNextStep();

    await consumerUsecasePage.selectLocationAndSource("Global Processing");
    await consumerUsecasePage.proceedToNextStep();

    await consumerUsecasePage.checkElementVisibility('.final-element');
  });

  test("Negative Case - Name too long", async () => {
    await consumerUsecasePage.addNewUseCase(
      consumerUsecasePage.generateRandomName('TooLongName') + 'asdfasdfasdfasdfasdfasdf', // long name
      '1000',
      'https://hyperdrive-dev.aexp.com/'
    );
    await consumerUsecasePage.multiSelectInput.click();
    await expect(page.locator('span:has-text("Name is too long")')).toBeVisible();
  });

  test("Negative Case - Value Exceeds Maximum", async () => {
    await consumerUsecasePage.addNewUseCase('babu44', '55556767676', 'https://hyperdrive-dev.aexp.com/');
    await consumerUsecasePage.multiSelectInput.click();
    await expect(page.locator('span:has-text("The value must be less than 10,000.")')).toBeVisible();
  });

  test("View and Edit Use Case", async () => {
    await consumerUsecasePage.viewUseCaseDetails();
    await expect(page.locator('h1:has-text("Use Case Details")')).toBeVisible();

    await page.locator('button:has-text("Edit")').click();
    await expect(page.locator('h1:has-text("Edit Use Case")')).toBeVisible();
  });

  test("Access Control Navigation and Request Access", async () => {
    await consumerUsecasePage.viewUseCaseDetails();
    await page.locator('#accessControlNavItem').click();
    await expect(page.locator('#requestAccess')).toBeVisible();
  });

  test("Verify Left Side Navigation Items", async () => {
    await consumerUsecasePage.viewUseCaseDetails();
    await page.locator('#accessControlNavItem').click();
    await expect(page.locator('button:has-text("Use Case Details")')).toBeVisible();
    await expect(page.locator('button:has-text("Request Access")')).toBeVisible();
  });
});
