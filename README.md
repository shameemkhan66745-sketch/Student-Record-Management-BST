#include <stdio.h>
#include <stdlib.h>

struct Student {
    int roll;
    char name[50];
    float marks;
};

struct Node {
    struct Student s;
    struct Node *left, *right;
};

// Create node
struct Node* create(struct Student s) {
    struct Node *n = malloc(sizeof(struct Node));
    n->s = s;
    n->left = n->right = NULL;
    return n;
}

// Insert student
struct Node* insert(struct Node *root, struct Student s) {
    if (root == NULL)
        return create(s);

    if (s.roll < root->s.roll)
        root->left = insert(root->left, s);
    else if (s.roll > root->s.roll)
        root->right = insert(root->right, s);
    else
        printf("Roll number already exists!\n");

    return root;
}

// Search student
struct Node* search(struct Node *root, int roll) {
    if (root == NULL || root->s.roll == roll)
        return root;

    if (roll < root->s.roll)
        return search(root->left, roll);

    return search(root->right, roll);
}

// Find minimum node
struct Node* minNode(struct Node *root) {
    while (root->left != NULL)
        root = root->left;
    return root;
}

// Delete student
struct Node* deleteNode(struct Node *root, int roll) {
    if (root == NULL)
        return NULL;

    if (roll < root->s.roll)
        root->left = deleteNode(root->left, roll);

    else if (roll > root->s.roll)
        root->right = deleteNode(root->right, roll);

    else {
        if (root->left == NULL) {
            struct Node *temp = root->right;
            free(root);
            return temp;
        }

        if (root->right == NULL) {
            struct Node *temp = root->left;
            free(root);
            return temp;
        }

        struct Node *temp = minNode(root->right);
        root->s = temp->s;
        root->right = deleteNode(root->right, temp->s.roll);
    }

    return root;
}

// Display students
void display(struct Node *root) {
    if (root != NULL) {
        display(root->left);
        printf("%d\t%s\t%.2f\n",
               root->s.roll, root->s.name,
               root->s.marks);
        display(root->right);
    }
}

// Save records
void save(struct Node *root, FILE *fp) {
    if (root != NULL) {
        save(root->left, fp);
        fwrite(&root->s, sizeof(struct Student), 1, fp);
        save(root->right, fp);
    }
}

// Load records
struct Node* load(struct Node *root) {
    FILE *fp = fopen("students.dat", "rb");
    struct Student s;

    if (fp == NULL)
        return root;

    while (fread(&s, sizeof(s), 1, fp) == 1)
        root = insert(root, s);

    fclose(fp);
    return root;
}

// Main function
int main() {
    struct Node *root = NULL, *temp;
    struct Student s;
    FILE *fp;
    int choice, roll;

    root = load(root);

    while (1) {
        printf("\n--- STUDENT MANAGEMENT SYSTEM ---\n");
        printf("1. Insert Student\n");
        printf("2. Search Student\n");
        printf("3. Delete Student\n");
        printf("4. Display Students\n");
        printf("5. Exit\n");
        printf("Enter choice: ");

        if (scanf("%d", &choice) != 1)
            break;

        switch (choice) {
        case 1:
            printf("Enter Roll Number: ");
            scanf("%d", &s.roll);

            printf("Enter Name: ");
            scanf(" %49[^\n]", s.name);

            printf("Enter Marks: ");
            scanf("%f", &s.marks);

            if (search(root, s.roll) != NULL)
                printf("Roll number already exists!\n");
            else {
                root = insert(root, s);
                printf("Student inserted!\n");
            }
            break;

        case 2:
            printf("Enter Roll Number: ");
            scanf("%d", &roll);

            temp = search(root, roll);

            if (temp != NULL)
                printf("Roll: %d\nName: %s\nMarks: %.2f\n",
                       temp->s.roll, temp->s.name,
                       temp->s.marks);
            else
                printf("Student not found!\n");
            break;

        case 3:
            printf("Enter Roll Number: ");
            scanf("%d", &roll);

            if (search(root, roll) == NULL)
                printf("Student not found!\n");
            else {
                root = deleteNode(root, roll);
                printf("Student deleted!\n");
            }
            break;

        case 4:
            printf("\nRoll\tName\tMarks\n");
            display(root);
            break;

        case 5:
            fp = fopen("students.dat", "wb");

            if (fp != NULL) {
                save(root, fp);
                fclose(fp);
            }

            printf("Records saved. Thank you!\n");
            return 0;

        default:
            printf("Invalid choice!\n");
        }
    }

    return 0;
}
